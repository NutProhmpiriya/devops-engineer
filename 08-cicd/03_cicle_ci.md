### CircleCI: สิ่งที่ต้องรู้เพื่อใช้งานให้มีประสิทธิภาพ

**CircleCI** เป็นระบบ Continuous Integration/Continuous Deployment (CI/CD) ที่ได้รับความนิยม ช่วยให้คุณสามารถทดสอบ สร้าง และปรับใช้งานแอปพลิเคชันได้อัตโนมัติ รองรับการทำงานบน Cloud หรือในเซิร์ฟเวอร์ของคุณเอง

---

## 1. **พื้นฐานของ CircleCI**
### 1.1 Key Concepts:
- **Pipeline**:
  - ชุดของงาน (Jobs) และ Workflow ที่ทำงานอัตโนมัติ
- **Jobs**:
  - งานเดี่ยวที่กำหนดคำสั่ง เช่น Build, Test, หรือ Deploy
- **Steps**:
  - คำสั่งที่กำหนดใน Job เช่น ติดตั้ง dependencies หรือรันเทสต์
- **Workflows**:
  - การจัดการลำดับการรัน Jobs
- **Executor**:
  - สิ่งแวดล้อมที่ใช้รัน Job เช่น Docker, VM, หรือ MacOS

---

## 2. **การตั้งค่า CircleCI**
### 2.1 ไฟล์ Config (`.circleci/config.yml`):
ไฟล์ `config.yml` เป็นหัวใจของ CircleCI ซึ่งกำหนดลำดับขั้นตอนและ Jobs

โครงสร้างพื้นฐาน:
```yaml
version: 2.1

executors:
  default:
    docker:
      - image: circleci/node:14

jobs:
  build:
    executor: default
    steps:
      - checkout
      - run: echo "Building the project"

workflows:
  version: 2
  build_and_test:
    jobs:
      - build
```

---

## 3. **Key Components**
### 3.1 Executors:
Executors เป็นตัวกำหนด environment ที่รัน Job:
- **Docker**:
  ใช้ Container สำหรับรันงาน:
  ```yaml
  executors:
    default:
      docker:
        - image: circleci/node:14
  ```
- **Machine**:
  ใช้ VM เต็มรูปแบบ:
  ```yaml
  executors:
    default:
      machine:
        image: ubuntu-2004:202101-01
  ```
- **MacOS**:
  ใช้สำหรับ iOS Development:
  ```yaml
  executors:
    default:
      macos:
        xcode: 12.5.1
  ```

### 3.2 Jobs:
Job คือหน่วยงานที่ประกอบด้วย Steps:
```yaml
jobs:
  build:
    docker:
      - image: circleci/python:3.9
    steps:
      - checkout
      - run: python --version
```

### 3.3 Steps:
ขั้นตอนที่ทำใน Job เช่น:
- **Checkout Code**:
  ```yaml
  - checkout
  ```
- **Run Commands**:
  ```yaml
  - run:
      name: Install Dependencies
      command: npm install
  ```

---

## 4. **การทำงานกับ Workflows**
Workflow ใช้จัดการลำดับของ Jobs:
```yaml
workflows:
  version: 2
  build_and_test:
    jobs:
      - build
      - test:
          requires:
            - build
```

- **Parallel Jobs**:
  Jobs รันพร้อมกัน:
  ```yaml
  workflows:
    version: 2
    parallel_jobs:
      jobs:
        - job1
        - job2
  ```

- **Sequential Jobs**:
  Jobs รันต่อเนื่อง:
  ```yaml
  workflows:
    version: 2
    sequential_jobs:
      jobs:
        - build
        - test:
            requires:
              - build
  ```

---

## 5. **การตั้งค่า Caching**
CircleCI รองรับการเก็บ Cache เพื่อลดเวลาในการ Build:
```yaml
steps:
  - restore_cache:
      keys:
        - v1-dependencies-{{ checksum "package-lock.json" }}
  - run: npm install
  - save_cache:
      paths:
        - node_modules
      key: v1-dependencies-{{ checksum "package-lock.json" }}
```

---

## 6. **Environment Variables**
### 6.1 ใช้ Environment Variables ใน Config:
```yaml
steps:
  - run:
      command: echo $MY_ENV_VAR
```

### 6.2 ตั้งค่าใน CircleCI Dashboard:
1. ไปที่ Settings > Environment Variables
2. เพิ่มตัวแปร เช่น `MY_ENV_VAR=example`

---

## 7. **การใช้งาน Orbs**
**Orbs** เป็นโมดูลสำเร็จรูปใน CircleCI:
- ตัวอย่าง Orbs สำหรับ Node.js:
  ```yaml
  version: 2.1
  orbs:
    node: circleci/node@4.7
  jobs:
    build:
      executor: node/default
      steps:
        - node/install-packages
  ```

- Orbs Marketplace: [CircleCI Orbs](https://circleci.com/developer/orbs)

---

## 8. **การ Debug Workflow**
- ใช้ SSH เพื่อ Debug:
  เปิดใช้งาน Debug Mode ใน CircleCI Dashboard และเชื่อมต่อด้วย SSH
- เพิ่ม Log ใน Steps:
  ```yaml
  - run:
      name: Debugging Step
      command: echo "Debugging this step"
  ```

---

## 9. **Best Practices**
1. **Pipeline as Code**:
   - เก็บไฟล์ `config.yml` ไว้ใน repository
2. **แยก Workflow ตามประเภทงาน**:
   - แยกงาน Build, Test, Deploy ออกจากกัน
3. **ใช้ Orbs**:
   - ลดความซับซ้อนด้วยโมดูลสำเร็จรูป
4. **เปิดใช้งาน Caching**:
   - เพิ่มความเร็วด้วยการเก็บ Cache
5. **ตั้งค่า Branch Protection**:
   - ใช้ CircleCI Pipeline ตรวจสอบโค้ดก่อน Merge
6. **รัน Jobs ใน Environment หลากหลาย**:
   ```yaml
   jobs:
     test:
       docker:
         - image: circleci/node:14
         - image: circleci/node:16
   ```

---

## 10. **ตัวอย่าง Workflow สำหรับ Docker**
Build และ Push Docker Image:
```yaml
version: 2.1

executors:
  docker-executor:
    docker:
      - image: docker:20.10.7

jobs:
  build:
    executor: docker-executor
    steps:
      - checkout
      - setup_remote_docker:
          version: 20.10.7
      - run:
          name: Build Docker Image
          command: docker build -t my-image .
      - run:
          name: Push Docker Image
          command: docker push my-image
```

---

😊