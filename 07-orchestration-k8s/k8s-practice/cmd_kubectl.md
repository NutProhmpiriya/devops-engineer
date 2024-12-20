| **คำสั่ง**                       | **ความหมาย**                                       | **การใช้งาน**                                | **ตัวอย่าง**                               |
|---------------------------------|----------------------------------------------------|--------------------------------------------|-----------------------------------------|
| `kubectl get nodes`             | แสดงรายการ nodes ทั้งหมดใน cluster      | ดู node ใน Kubernetes cluster                 | `kubectl get nodes`                     |
| `kubectl get pods`              | แสดงรายการ pods ทั้งหมดใน namespace ปัจจุบัน | ดูสถานะ pods ใน cluster               | `kubectl get pods`                      |
| `kubectl get services`          | แสดงรายการ services ทั้งหมดใน namespace ปัจจุบัน | ตรวจสอบ service ใน cluster           | `kubectl get services`                  |
| `kubectl describe pod <pod-name>` | ดูรายละเอียดของ pod ที่กำหนด   | ดูสถานะ, Event และรายละเอียด pod | `kubectl describe pod my-pod`          |
| `kubectl logs <pod-name>`       | ดู log ของ pod ที่กำหนด                      | ตรวจสอบ log สำหรับ debug         | `kubectl logs my-pod`                   |
| `kubectl exec -it <pod-name> -- /bin/bash` | เข้าไปที่ shell ของ container ใน pod | เข้าไปทำงานภายใน container | `kubectl exec -it my-pod -- /bin/bash` |
| `kubectl apply -f <file.yaml>`  | Deploy หรือ update resource จากไฟล์ YAML       | ใช้ในการ deploy resource                  | `kubectl apply -f deployment.yaml`      |
| `kubectl delete -f <file.yaml>` | ลบ resource ที่สร้างจาก YAML               | เพื่อลบ resource ที่สร้างไว้ | `kubectl delete -f deployment.yaml`     |
| `kubectl scale deployment <deployment-name> --replicas=<n>` | ปรับจำนวน replicas ของ deployment      | เพิ่มและลดจำนวน pod       | `kubectl scale deployment my-deployment --replicas=3` |
| `kubectl config get-contexts`   | แสดง context ทั้งหมดใน kubeconfig         | เพื่อเปลี่ยนเปลี่น cluster | `kubectl config get-contexts`           |
