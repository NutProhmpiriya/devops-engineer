### GitHub Actions: สิ่งที่ต้องรู้และใช้งานได้อย่างมีประสิทธิภาพ

**GitHub Actions** เป็นระบบ CI/CD (Continuous Integration/Continuous Deployment) ที่มีอยู่ใน GitHub ซึ่งช่วยให้คุณสามารถสร้าง workflow อัตโนมัติสำหรับการทดสอบ สร้าง และปรับใช้งานแอปพลิเคชันได้อย่างง่ายดายโดยตรงจาก GitHub Repository ของคุณ

---

## 1. **พื้นฐาน GitHub Actions**
### 1.1 Key Concepts:
- **Workflow**:
  - เป็นชุดของงาน (jobs) ที่ทำงานอัตโนมัติ
  - เขียนในไฟล์ YAML ภายใต้โฟลเดอร์ `.github/workflows/`
- **Jobs**:
  - แต่ละงานใน workflow ประกอบด้วยหลายๆ **steps**
- **Steps**:
  - การดำเนินการที่ทำใน job เช่น รันคำสั่งหรือใช้งาน action
- **Actions**:
  - คำสั่งหรือโปรเจกต์สำเร็จรูปที่ใช้ซ้ำใน workflow

### 1.2 Trigger Events:
Workflow จะเริ่มทำงานเมื่อเกิด event ที่กำหนดไว้ เช่น:
- `push`: เมื่อ push โค้ดเข้า repository
- `pull_request`: เมื่อสร้างหรืออัปเดต pull request
- `schedule`: รันตามเวลาที่กำหนด (cron)
- `workflow_dispatch`: เรียกใช้งาน workflow ด้วยมือ

ตัวอย่าง trigger:
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  schedule:
    - cron: "0 12 * * *"
```

---

## 2. **โครงสร้าง Workflow (YAML)**
ตัวอย่าง workflow พื้นฐาน:
```yaml
name: CI Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 16

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

### อธิบาย:
1. **name**: ตั้งชื่อ workflow
2. **on**: กำหนด event ที่ trigger workflow
3. **jobs**:
   - **runs-on**: กำหนด environment ที่จะรัน เช่น `ubuntu-latest`, `windows-latest`, `macos-latest`
   - **steps**: ขั้นตอนการทำงานใน job
     - ใช้ `uses` เพื่อเรียกใช้ action
     - ใช้ `run` เพื่อรันคำสั่ง shell

---

## 3. **การใช้ Secrets และ Environment Variables**
### 3.1 Secrets:
- ใช้สำหรับเก็บข้อมูลที่เป็นความลับ เช่น API keys
- ตั้งค่าใน GitHub Settings > Secrets
- ใช้ใน workflow:
  ```yaml
  steps:
    - name: Deploy
      run: echo $MY_SECRET
      env:
        MY_SECRET: ${{ secrets.MY_SECRET }}
  ```

### 3.2 Environment Variables:
- กำหนดตัวแปรที่ใช้ใน workflow:
  ```yaml
  jobs:
    build:
      env:
        NODE_ENV: production
      steps:
        - name: Print ENV
          run: echo $NODE_ENV
  ```

---

## 4. **การจัดการ Workflow ขนาดใหญ่**
### 4.1 การแบ่งงานด้วย Jobs:
- รัน jobs หลายงานพร้อมกันหรือแบบลำดับ:
  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - run: echo "Building..."
    
    test:
      runs-on: ubuntu-latest
      needs: build
      steps:
        - run: echo "Testing..."
  ```

### 4.2 Reusable Workflows:
- สร้าง workflow ที่เรียกใช้ซ้ำได้:
  ```yaml
  on:
    workflow_call:
      inputs:
        environment:
          required: true
          type: string

  jobs:
    reusable-job:
      runs-on: ubuntu-latest
      steps:
        - run: echo "Environment: ${{ inputs.environment }}"
  ```

เรียกใช้งานจาก workflow อื่น:
```yaml
jobs:
  call-reusable:
    uses: owner/repo/.github/workflows/reusable-workflow.yml@main
    with:
      environment: production
```

---

## 5. **GitHub Marketplace**
GitHub มี **Marketplace** ที่รวม Actions สำเร็จรูป เช่น:
- **actions/checkout**: ดึงโค้ดจาก repository
- **actions/setup-node**: ตั้งค่า Node.js
- **docker/build-push-action**: Build และ Push Docker Image
- **aws-actions/configure-aws-credentials**: เชื่อมต่อกับ AWS

ค้นหาเพิ่มเติม: [GitHub Marketplace](https://github.com/marketplace)

---

## 6. **การใช้งาน Docker ใน GitHub Actions**
ใช้ Docker container ใน workflow:
```yaml
jobs:
  docker-job:
    runs-on: ubuntu-latest
    container:
      image: node:16
    steps:
      - name: Run in Docker
        run: node -v
```

---

## 7. **การ Debug Workflow**
- ใช้คำสั่ง `echo` หรือ `run` เพื่อพิมพ์ค่าตัวแปร
- ดู Log โดยคลิกที่ workflow ใน GitHub Actions UI
- ใช้ Debug mode:
  ```yaml
  steps:
    - name: Enable debug logging
      run: echo "::debug::This is a debug message"
  ```

---

## 8. **แนวทางปฏิบัติที่ดี (Best Practices)**
1. **Pipeline as Code**:
   - เก็บ workflow ไว้ใน repository เดียวกับโค้ด
2. **แยก Workflow ตามประเภทงาน**:
   - สร้าง workflow แยกสำหรับ build, test, และ deploy
3. **ใช้ Reusable Workflows**:
   - ลดการทำงานซ้ำใน workflow ขนาดใหญ่
4. **จัดการ Secrets อย่างปลอดภัย**:
   - ใช้ Secrets Manager ของ GitHub
5. **เปิดใช้งาน Branch Protection**:
   - ผสาน workflow กับ branch protection rule เช่นการตรวจสอบ CI ก่อน merge
6. **Run Tests ใน Environment ที่หลากหลาย**:
   ```yaml
   strategy:
     matrix:
       node-version: [12, 14, 16]
   steps:
     - name: Set up Node.js
       uses: actions/setup-node@v3
       with:
         node-version: ${{ matrix.node-version }}
   ```

