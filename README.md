<h1 align="center">
  <span style="background: linear-gradient(to right, #1e3c72, #2a5298); -webkit-background-clip: text; color: transparent;">
    🐔 Chicken-Disease Project
  </span>
</h1>

<p align="center">
  <strong>Detect common chicken diseases using Deep Learning & deploy via AWS CI/CD</strong>
</p>

<p align="center">
  <a href="https://github.com/rayhanhcse/Chicken-Disease">
    <img src="https://img.shields.io/badge/GitHub-rayhanhcse-181717?style=for-the-badge&logo=github" alt="GitHub Badge"/>
  </a>
  <a href="mailto:rayhanhcse@gmail.com">
    <img src="https://img.shields.io/badge/Email-rayhanhcse@gmail.com-D14836?style=for-the-badge&logo=gmail" alt="Email Badge"/>
  </a>
  <a href="https://linkedin.com/in/rayhanchse">
    <img src="https://img.shields.io/badge/LinkedIn-rayhanchse-0077B5?style=for-the-badge&logo=linkedin" alt="LinkedIn Badge"/>
  </a>
  <img src="https://img.shields.io/badge/Python-3.8-3776AB?style=for-the-badge&logo=python" alt="Python Badge"/>
  <img src="https://img.shields.io/badge/Anaconda-202020?style=for-the-badge&logo=anaconda" alt="Anaconda Badge"/>
</p>

---


## 🌐 Dataset

**Data Link:** [📦 Download Dataset](https://drive.google.com/file/d/1pV0DAdyjzsjk0HL7f8_5qiS_mVyjYk25/view?usp=sharing)

---

## 🧠 Workflows

1. Update `config.yaml`
2. Update `params.yaml`
3. Update the entity
4. Update the configuration manager in `src/config`
5. Update the components
6. Update the pipeline
7. Update `main.py`
8. Update `app.py`

---

## ⚙️ How to Run

### 1️⃣ Clone Repository

```bash
git clone https://github.com/rayhanhcse/Chicken-Disease.git
cd Chicken-Disease
```

### 2️⃣ Create Conda Environment

```bash
conda create -n chicken python=3.8 -y
conda activate chicken
```

### 3️⃣ Install Dependencies

```bash
pip install -r requirements.txt
```

### 4️⃣ Export AWS Credentials (Optional)

```bash
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_ACCESS_KEY"
```

### 5️⃣ Run Application

```bash
python app.py
```

Now open your local host and port in browser.

---

## ☁️ AWS CI/CD Deployment (with GitHub Actions)

### Step 1: Login to AWS Console

### Step 2: Create IAM User

**Permissions:**

* EC2 Access (Virtual Machine)
* ECR Access (Elastic Container Registry)

**Policies:**

* `AmazonEC2ContainerRegistryFullAccess`
* `AmazonEC2FullAccess`

### Step 3: Create ECR Repository

```text
315865595366.dkr.ecr.us-east-1.amazonaws.com/chicken
```

### Step 4: Create EC2 Instance (Ubuntu)

### Step 5: Install Docker

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker
```

### Step 6: Configure EC2 as Self‑Hosted Runner

`GitHub → Settings → Actions → Runner → New Self-Hosted Runner`

### Step 7: Setup GitHub Secrets

| Secret Key            | Example                                       |
| --------------------- | --------------------------------------------- |
| AWS_ACCESS_KEY_ID     | YOUR_ACCESS_KEY_ID                            |
| AWS_SECRET_ACCESS_KEY | YOUR_SECRET_ACCESS_KEY                        |
| AWS_REGION            | us-east-1                                     |
| AWS_ECR_LOGIN_URI     | 566373416292.dkr.ecr.ap-south-1.amazonaws.com |
| ECR_REPOSITORY_NAME   | chicken                                       |

---

## 🐳 Docker Deployment

```bash
# Login to ECR
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ECR_LOGIN_URI

# Build and push image
docker build -t chicken .
docker tag chicken:latest $AWS_ECR_LOGIN_URI/chicken:latest
docker push $AWS_ECR_LOGIN_URI/chicken:latest
```

### On EC2:

```bash
docker pull $AWS_ECR_LOGIN_URI/chicken:latest
docker run -d -p 80:5000 $AWS_ECR_LOGIN_URI/chicken:latest
```

---

## 🚀 GitHub Actions (Workflow Example)

```yaml
name: CI/CD - Deploy to AWS
on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
      - name: Login to ECR
        run: aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | docker login --username AWS --password-stdin ${{ secrets.AWS_ECR_LOGIN_URI }}
      - name: Build, Tag & Push Image
        run: |
          docker build -t ${{ secrets.ECR_REPOSITORY_NAME }}:latest .
          docker tag ${{ secrets.ECR_REPOSITORY_NAME }}:latest ${{ secrets.AWS_ECR_LOGIN_URI }}/${{ secrets.ECR_REPOSITORY_NAME }}:latest
          docker push ${{ secrets.AWS_ECR_LOGIN_URI }}/${{ secrets.ECR_REPOSITORY_NAME }}:latest
```

---

## 💡 Tips & Troubleshooting

* **Port already in use?** → Change the port or kill existing process.
* **Docker not found?** → Reconnect your EC2 session after `usermod`.
* **ECR login errors?** → Verify AWS credentials and region.

---

## 👨‍💻 Author

**Rayhan Hussain**
📧 [LinkedIn Profile](https://linkedin.com/in/rayhanchse)
🐙 [GitHub](https://github.com/rayhanhcse)

---

### 🌈 *Made with ❤️ by Rayhan Hussain — turning code into innovation!*
