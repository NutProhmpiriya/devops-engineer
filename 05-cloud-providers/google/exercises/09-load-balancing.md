# แบบฝึกหัดที่ 9: การตั้งค่า Load Balancing
# Exercise 9: Setting up Load Balancing

## โจทย์ | Problem
ตั้งค่าระบบ Load Balancing สำหรับแอปพลิเคชันที่ต้องรองรับการใช้งานระดับโลก โดยมีความต้องการดังนี้:

1. ตั้งค่า:
   - Global HTTP(S) Load Balancer
   - SSL certificates
   - URL maps for different services
   - Backend services with health checks

2. กำหนด:
   - Custom domains
   - CDN configuration
   - Traffic splitting
   - Auto scaling

## เฉลย | Solution

### 1. สร้าง Instance Groups:
```bash
# Create instance template
gcloud compute instance-templates create web-template \
    --machine-type=e2-medium \
    --image-family=debian-10 \
    --image-project=debian-cloud \
    --tags=http-server \
    --metadata-from-file=startup-script=startup.sh

# Create managed instance group
gcloud compute instance-groups managed create web-group \
    --template=web-template \
    --size=2 \
    --zone=asia-southeast1-a \
    --health-check=http-basic-check \
    --initial-delay=300

# Configure autoscaling
gcloud compute instance-groups managed set-autoscaling web-group \
    --zone=asia-southeast1-a \
    --max-num-replicas=5 \
    --target-cpu-utilization=0.75
```

### 2. สร้าง Health Checks:
```bash
# Create health check
gcloud compute health-checks create http http-basic-check \
    --port=80 \
    --check-interval=30s \
    --timeout=10s \
    --healthy-threshold=2 \
    --unhealthy-threshold=3 \
    --request-path=/health
```

### 3. ตั้งค่า SSL Certificates:
```bash
# Create SSL certificate
gcloud compute ssl-certificates create www-cert \
    --domains=www.example.com \
    --global

# Upload custom certificate
gcloud compute ssl-certificates create custom-cert \
    --certificate=cert.pem \
    --private-key=key.pem \
    --global
```

### 4. สร้าง URL Maps:
```bash
# Create backend service
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

# Add instance group to backend service
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=web-group \
    --instance-group-zone=asia-southeast1-a \
    --global

# Create URL map
gcloud compute url-maps create web-map \
    --default-service=web-backend-service

# Create path matcher for different services
gcloud compute url-maps add-path-matcher web-map \
    --path-matcher-name=service-paths \
    --default-service=web-backend-service \
    --path-rules="/api/*=api-backend-service,/static/*=static-backend-service"
```

### 5. ตั้งค่า Frontend Configuration:
```bash
# Create target HTTP proxy
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map=web-map

# Create target HTTPS proxy
gcloud compute target-https-proxies create https-lb-proxy \
    --url-map=web-map \
    --ssl-certificates=www-cert

# Create global forwarding rules
gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4 \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80

gcloud compute forwarding-rules create https-content-rule \
    --address=lb-ipv4 \
    --global \
    --target-https-proxy=https-lb-proxy \
    --ports=443
```

### 6. ตั้งค่า CDN:
```bash
# Enable Cloud CDN
gcloud compute backend-services update web-backend-service \
    --enable-cdn \
    --global

# Configure cache key
gcloud compute backend-services update web-backend-service \
    --global \
    --cache-mode=USE_ORIGIN_HEADERS \
    --cache-key-policy-include-host \
    --cache-key-policy-include-protocol \
    --cache-key-policy-include-query-string
```

### 7. ตั้งค่า Traffic Splitting:
```bash
# Create new version of backend
gcloud compute backend-services create web-backend-service-v2 \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

# Update URL map for traffic splitting
gcloud compute url-maps add-host-rule web-map \
    --hosts=www.example.com \
    --path-matcher-name=traffic-split

gcloud compute url-maps add-path-matcher web-map \
    --default-service=web-backend-service \
    --path-matcher-name=traffic-split \
    --route-rules="priority=1,service=web-backend-service-v2,weight=0.2"
```

## การทดสอบ | Testing
```bash
# Test health checks
gcloud compute health-checks describe http-basic-check

# Test backend services
gcloud compute backend-services get-health web-backend-service \
    --global

# Test URL map
gcloud compute url-maps validate web-map

# Test SSL certificate
gcloud compute ssl-certificates describe www-cert \
    --global

# Monitor traffic
gcloud monitoring metrics list \
    --filter="metric.type=loadbalancing.googleapis.com/https/request_count"
```

## เพิ่มเติม | Additional Notes
- ใช้ Cloud Armor สำหรับ DDoS protection
- ตั้งค่า custom headers สำหรับ security
- พิจารณาใช้ serverless NEG สำหรับ Cloud Run
- ตั้งค่า logging และ monitoring
- ใช้ IAP สำหรับ internal applications
