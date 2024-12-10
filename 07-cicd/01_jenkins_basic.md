การใช้งาน **Jenkins** อย่างมีประสิทธิภาพ จำเป็นต้องเข้าใจพื้นฐานและองค์ประกอบที่สำคัญ รวมถึงการตั้งค่าและการจัดการ Jenkins Pipeline เพื่อรองรับการทำงานในระบบ CI/CD (Continuous Integration/Continuous Deployment) นี่คือสิ่งที่ควรรู้:

---

## 1. **พื้นฐานของ Jenkins**
### 1.1 Jenkins คืออะไร?
- เป็นเครื่องมือ Open-source สำหรับ CI/CD ที่ช่วย:
  - รันการ build และ deploy อัตโนมัติ
  - จัดการ workflow ของทีมพัฒนา
  - ผนวกเข้ากับ tools อื่น ๆ เช่น Git, Docker, Kubernetes ฯลฯ

### 1.2 แนวคิดสำคัญ:
- **Job/Project**: หน่วยของงานที่ Jenkins รัน เช่น build, test หรือ deploy
- **Build**: กระบวนการทำงานของ Jenkins เช่น การคอมไพล์โค้ดหรือรันสคริปต์
- **Node/Agent**: เครื่องที่รันงาน (อาจเป็นเครื่อง Jenkins Master เองหรือ Agent ที่เชื่อมต่อ)
- **Pipeline**: กำหนดขั้นตอน CI/CD ในรูปแบบของโค้ด (Declarative หรือ Scripted Pipeline)

---

## 2. **การติดตั้ง Jenkins**
### 2.1 วิธีการติดตั้ง
- **Docker**:
  ```bash
  docker run -d -p 8080:8080 -p 50000:50000 --name jenkins jenkins/jenkins:lts
  ```
- **Linux/Windows**:
  - ติดตั้งจากแพ็กเกจ `.war` หรือแพ็กเกจที่รองรับ
- การตั้งค่าเริ่มต้น: ใช้รหัสผ่านที่สร้างในไฟล์ `initialAdminPassword`

### 2.2 Plugins ที่จำเป็น:
- **Git Plugin**: สำหรับเชื่อมต่อกับ Git repositories
- **Pipeline Plugin**: สำหรับเขียน Jenkins Pipeline
- **Docker Plugin**: สำหรับใช้งาน Docker
- **Kubernetes Plugin**: สำหรับผสาน Jenkins กับ Kubernetes

---

## 3. **การตั้งค่า Jenkins**
### 3.1 การสร้าง Job
- เลือกประเภทงาน เช่น Freestyle Project หรือ Pipeline
- ตั้งค่า Repository URL (เช่น GitHub หรือ GitLab)
- กำหนด Trigger เช่น:
  - **Poll SCM**: ตรวจโค้ดใน Git repository เป็นระยะ
  - **Webhook**: ใช้สำหรับอัปเดต Jenkins แบบเรียลไทม์

### 3.2 การตั้งค่า Agent/Node
- เพิ่ม Jenkins Agent สำหรับรันงาน (ลดภาระ Jenkins Master)
- ใช้ SSH, Docker, หรือ Kubernetes ในการจัดการ Agent

---

## 4. **Jenkins Pipeline**
Jenkins Pipeline คือ **โค้ด** ที่กำหนด workflow CI/CD โดยเขียนในไฟล์ `Jenkinsfile`

### 4.1 รูปแบบ Pipeline
#### Declarative Pipeline (อ่านง่าย):
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

#### Scripted Pipeline (ยืดหยุ่น):
```groovy
node {
    stage('Build') {
        echo 'Building...'
    }
    stage('Test') {
        echo 'Testing...'
    }
    stage('Deploy') {
        echo 'Deploying...'
    }
}
```

### 4.2 ฟีเจอร์สำคัญใน Pipeline:
- **Parallel Stages**: รันงานพร้อมกันในขั้นตอนเดียว
  ```groovy
  stages {
      stage('Parallel Tasks') {
          parallel {
              stage('Task 1') {
                  steps {
                      echo 'Running Task 1'
                  }
              }
              stage('Task 2') {
                  steps {
                      echo 'Running Task 2'
                  }
              }
          }
      }
  }
  ```
- **Environment Variables**:
  กำหนดตัวแปรใน Pipeline:
  ```groovy
  environment {
      MY_VAR = 'Hello World'
  }
  steps {
      echo "Variable: ${env.MY_VAR}"
  }
  ```

---

## 5. **การผสาน Jenkins กับ Tools อื่น**
### 5.1 Git Integration
- เชื่อมต่อ Jenkins กับ GitHub/GitLab โดยใช้ Webhook หรือ Poll SCM
- ตัวอย่างใน Pipeline:
  ```groovy
  pipeline {
      agent any
      stages {
          stage('Checkout') {
              steps {
                  git branch: 'main', url: 'https://github.com/user/repo.git'
              }
          }
      }
  }
  ```

### 5.2 Docker Integration
- ใช้ Docker สำหรับ build และ deploy:
  ```groovy
  pipeline {
      agent {
          docker {
              image 'maven:3.6.3'
          }
      }
      stages {
          stage('Build') {
              steps {
                  sh 'mvn clean install'
              }
          }
      }
  }
  ```

### 5.3 Kubernetes Integration
- ใช้ Kubernetes Plugin เพื่อ deploy แอปพลิเคชัน:
  ```groovy
  pipeline {
      agent {
          kubernetes {
              yaml '''
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: jnlp
                  image: jenkins/inbound-agent
              '''
          }
      }
      stages {
          stage('Deploy') {
              steps {
                  echo 'Deploying to Kubernetes'
              }
          }
      }
  }
  ```

---

## 6. **การแก้ปัญหาและ Debug**
- ดู Log ใน Jenkins UI
- ใช้คำสั่ง `echo` หรือ `sh` ใน Pipeline เพื่อ Debug
- ตรวจสอบการเชื่อมต่อ Jenkins Agent กับ Master

---

## 7. **แนวทางปฏิบัติที่ดี**
1. ใช้ **Pipeline as Code**: เก็บ Jenkinsfile ไว้ใน Git repository
2. แยก Jenkins Master และ Agent: ลดภาระงานหนักบน Master
3. ใช้ Jenkins Shared Library: ลดการทำงานซ้ำโดยสร้างฟังก์ชันส่วนกลาง
4. อัปเดต Jenkins และ Plugins เป็นประจำเพื่อความปลอดภัย
5. ใช้ Credential Management สำหรับจัดการรหัสผ่านหรือ Token

---

## 8. **สิ่งที่ควรศึกษาเพิ่มเติม**
- การใช้ Jenkins Blue Ocean (UI ที่อ่านง่าย)
- การตั้งค่า Jenkins High Availability (HA) สำหรับระบบขนาดใหญ่
- การตั้งค่าการตรวจสอบ (Monitoring) Jenkins ด้วย Prometheus และ Grafana
- Jenkinsfile ขั้นสูง เช่น การกำหนด Retry หรือ Timeout:
  ```groovy
  options {
      timeout(time: 1, unit: 'HOURS')
      retry(3)
  }
  ```

