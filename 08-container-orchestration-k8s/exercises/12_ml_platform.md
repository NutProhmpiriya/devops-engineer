# Exercise 12: Machine Learning Platform on Kubernetes

## Task
สร้าง Machine Learning Platform บน Kubernetes โดยใช้ Kubeflow และ MLflow

### Requirements
1. ติดตั้ง Kubeflow
2. Set up MLflow Tracking Server
3. Configure Distributed Training
4. Implement Model Serving
5. Set up Pipeline Orchestration
6. Configure Auto-scaling for Training Jobs

## Solution

1. Kubeflow Installation (`kubeflow-install.yaml`):
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- github.com/kubeflow/manifests/common/kubeflow-namespace/base
- github.com/kubeflow/manifests/common/istio-1-9/kubeflow-istio-resources/base
- github.com/kubeflow/manifests/apps/pipeline/upstream/env/platform-agnostic-multi-user
- github.com/kubeflow/manifests/apps/katib/upstream/installs/katib-with-kubeflow
- github.com/kubeflow/manifests/apps/centraldashboard/upstream/overlays/istio
namespace: kubeflow
```

2. MLflow Tracking Server (`mlflow.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mlflow-tracking
  namespace: mlflow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mlflow
  template:
    metadata:
      labels:
        app: mlflow
    spec:
      containers:
      - name: mlflow
        image: ghcr.io/mlflow/mlflow:v2.3.0
        args:
        - mlflow
        - server
        - --backend-store-uri=postgresql://mlflow@postgresql:5432/mlflow
        - --default-artifact-root=s3://mlflow/artifacts
        - --host=0.0.0.0
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: mlflow-s3
              key: access-key
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: mlflow-s3
              key: secret-key
---
apiVersion: v1
kind: Service
metadata:
  name: mlflow
  namespace: mlflow
spec:
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app: mlflow
```

3. Distributed Training Job (`distributed-training.yaml`):
```yaml
apiVersion: "kubeflow.org/v1"
kind: PyTorchJob
metadata:
  name: pytorch-dist
  namespace: kubeflow
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: pytorch
            image: pytorch-training:latest
            resources:
              limits:
                nvidia.com/gpu: 1
            env:
            - name: MLFLOW_TRACKING_URI
              value: http://mlflow.mlflow:5000
    Worker:
      replicas: 4
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: pytorch
            image: pytorch-training:latest
            resources:
              limits:
                nvidia.com/gpu: 1
            env:
            - name: MLFLOW_TRACKING_URI
              value: http://mlflow.mlflow:5000
```

4. Model Serving (`model-serving.yaml`):
```yaml
apiVersion: "serving.kubeflow.org/v1beta1"
kind: "InferenceService"
metadata:
  name: "resnet50"
  namespace: kubeflow
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 3
    scaleTarget: 70
    tensorflow:
      storageUri: "s3://mlflow/artifacts/models/resnet50"
      resources:
        limits:
          memory: 4Gi
          nvidia.com/gpu: 1
        requests:
          memory: 2Gi
    serviceAccountName: model-serving-sa
```

5. Pipeline Orchestration (`pipeline.yaml`):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: ml-training-pipeline-
  namespace: kubeflow
spec:
  entrypoint: ml-pipeline
  templates:
  - name: ml-pipeline
    dag:
      tasks:
      - name: data-preprocessing
        template: data-prep
      - name: model-training
        template: training
        dependencies: [data-preprocessing]
      - name: model-evaluation
        template: evaluation
        dependencies: [model-training]
      - name: model-serving
        template: deploy
        dependencies: [model-evaluation]
  
  - name: data-prep
    container:
      image: data-prep:latest
      command: [python, preprocess.py]
      env:
      - name: MLFLOW_TRACKING_URI
        value: http://mlflow.mlflow:5000
  
  - name: training
    resource:
      action: create
      manifest: |
        apiVersion: "kubeflow.org/v1"
        kind: PyTorchJob
        metadata:
          generateName: pytorch-training-
        spec:
          pytorchReplicaSpecs:
            Master:
              replicas: 1
              template:
                spec:
                  containers:
                  - name: pytorch
                    image: training:latest
  
  - name: evaluation
    container:
      image: model-eval:latest
      command: [python, evaluate.py]
      env:
      - name: MLFLOW_TRACKING_URI
        value: http://mlflow.mlflow:5000
  
  - name: deploy
    resource:
      action: create
      manifest: |
        apiVersion: "serving.kubeflow.org/v1beta1"
        kind: "InferenceService"
        metadata:
          generateName: model-serving-
        spec:
          predictor:
            tensorflow:
              storageUri: "s3://mlflow/artifacts/models/latest"
```

6. Auto-scaling Configuration (`autoscaling.yaml`):
```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: training-autoscaler
  namespace: kubeflow
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: training-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

### การติดตั้งและ Configuration

1. ติดตั้ง Kubeflow:
```bash
# Install Kubeflow
kustomize build kubeflow-install.yaml | kubectl apply -f -

# Verify installation
kubectl get pods -n kubeflow
```

2. Set up MLflow:
```bash
# Create namespace
kubectl create namespace mlflow

# Create secrets
kubectl create secret generic mlflow-s3 \
  --from-literal=access-key=$AWS_ACCESS_KEY_ID \
  --from-literal=secret-key=$AWS_SECRET_ACCESS_KEY \
  -n mlflow

# Deploy MLflow
kubectl apply -f mlflow.yaml
```

3. Configure GPU Support:
```bash
# Install NVIDIA device plugin
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/master/nvidia-device-plugin.yml

# Verify GPU nodes
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
```

### การทดสอบ

1. Run Training Job:
```bash
# Submit training job
kubectl apply -f distributed-training.yaml

# Monitor training progress
kubectl logs -f pytorch-dist-master-0 -n kubeflow
```

2. Test Model Serving:
```bash
# Deploy model
kubectl apply -f model-serving.yaml

# Test inference
curl -X POST http://resnet50.kubeflow/v1/models/resnet50:predict \
  -d @./test-image.json
```

3. Run Pipeline:
```bash
# Submit pipeline
argo submit pipeline.yaml

# Monitor pipeline progress
argo list
argo get @latest
```

### การ Monitor

1. MLflow Tracking:
```bash
# Port forward MLflow UI
kubectl port-forward svc/mlflow 5000:5000 -n mlflow

# Access UI at http://localhost:5000
```

2. Kubeflow Dashboard:
```bash
# Access dashboard
kubectl port-forward svc/centraldashboard 8080:80 -n kubeflow
```

### Tips
- Use persistent volumes for training data
- Implement proper resource quotas
- Monitor GPU utilization
- Regular backup of MLflow database
- Version control for models
- Implement A/B testing for models
- Monitor inference latency
