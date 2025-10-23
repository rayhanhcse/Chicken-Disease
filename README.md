<h1 align="center">🎯 <span style="background:linear-gradient(90deg,#ff7eb3,#81f7d8);-webkit-background-clip:text;color:transparent;">Chicken‑Disease</span></h1>

<p align="center">An end‑to‑end deep learning project to detect common chicken diseases with a web app and AWS CI/CD deployment 🚀</p>

---

<p align="center">
  <a href="#features">✨ Features</a> ·
  <a href="#quick-start">⚡ Quick Start</a> ·
  <a href="#aws-cicd">☁️ AWS CI/CD</a> ·
  <a href="#contributing">🤝 Contributing</a>
</p>

---

## 🔥 Highlights

* End-to-end pipeline: data → preprocessing → training → evaluation → deployment
* Simple web app (`app.py`) to run inference locally
* Dockerized for reproducible deployment
* GitHub Actions → ECR → EC2 self-hosted runner CI/CD flow
* Config-driven (`config.yaml`, `params.yaml`) for easy experimentation

## 📁 Repository

```
Chicken-Disease/
├─ src/
├─ config.yaml
├─ params.yaml
├─ requirements.txt
├─ app.py
├─ main.py
├─ Dockerfile
└─ README.md
```

## 📥 Data

**Data Link:** [Download dataset (Google Drive)](https://drive.google.com/file/d/1pV0DAdyjzsjk0HL7f8_5qiS_mVyjYk25/view?usp=sharing)

> Place the downloaded data inside `data/` (make `data/raw` if not present).

---

## 🧭 Table of Contents

1. [Quick Start](#quick-start)
2. [Configuration](#configuration)
3. [Project Structure & Workflows](#workflows)
4. [Local Run](#local-run)
5. [Docker & AWS CI/CD](#aws-cicd)
6. [Tips & Troubleshooting](#tips)
7. [Contributing](#contributing)
8. [License & Contact](#license)

---

## ⚡ Quick Start

**1. Clone**

```bash
git clone https://github.com/rayhanhcse/Chicken-Disease.git
cd Chicken-Disease
```

**2. Create conda env**

```bash
conda create -n chicken python=3.8 -y
conda activate chicken
pip install -r requirements.txt
```

**3. Update configs**

* `config.yaml` — pipeline and path settings
* `params.yaml` — training hyperparameters

**4. Export (optional) AWS creds for CI/CD**

```bash
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_ACCESS_KEY"
export AWS_REGION="us-east-1"
```

**5. Run app**

```bash
python app.py
```

Open your browser at `http://127.0.0.1:5000` (or the printed host/port).

---

## 🛠 Configuration

* `config.yaml` — central configuration manager. Keep paths and artifact locations here.
* `params.yaml` — training hyperparameters (epochs, batch_size, lr, augmentations).
* `src/config/*` — code that reads these files and builds objects used by components.

**Recommended change flow**

1. Update `config.yaml`
2. Update `params.yaml`
3. Update `src/entity` (dataclasses)
4. Update `src/config/configuration_manager.py`
5. Update components in `src/components/` (data, model, trainer)
6. Update `src/pipeline.py` to orchestrate
7. Run `main.py` or `app.py` for web

---

## 🔁 Workflows (short)

1. Update `config.yaml`
2. Update `params.yaml`
3. Update domain entities (`src/entity`)
4. Update configuration manager (`src/config`)
5. Implement/modify components (`src/components`)
6. Update pipeline (`src/pipeline.py`)
7. Update `main.py` for CLI/train runs and `app.py` for the web app

---

## 🐳 Docker & AWS CI/CD (modernized)

### Overview

1. Build Docker image locally or in CI
2. Push image to AWS ECR
3. Deploy to EC2 (self-hosted runner) or ECS
4. Use GitHub Actions to automate build → push → deploy

### Minimal IAM policies for deploy user

* `AmazonEC2ContainerRegistryFullAccess`
* `AmazonEC2FullAccess`

**NOTE:** For production tighten permissions to only the resources needed.

### Recommended secrets (GitHub)

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_REGION` (e.g. `us-east-1`)
* `AWS_ECR_LOGIN_URI` (e.g. `123456789012.dkr.ecr.us-east-1.amazonaws.com`)
* `ECR_REPOSITORY_NAME` (e.g. `chicken`)

### Quick commands (EC2)

```bash
# install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker

# login to ECR (from CI or local with awscli)
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ECR_LOGIN_URI

# build and push
docker build -t $ECR_REPOSITORY_NAME:latest .
docker tag $ECR_REPOSITORY_NAME:latest $AWS_ECR_LOGIN_URI/$ECR_REPOSITORY_NAME:latest
docker push $AWS_ECR_LOGIN_URI/$ECR_REPOSITORY_NAME:latest

# on EC2
docker pull $AWS_ECR_LOGIN_URI/$ECR_REPOSITORY_NAME:latest
docker run -d --restart always -p 80:5000 $AWS_ECR_LOGIN_URI/$ECR_REPOSITORY_NAME:latest
```

### GitHub Actions (example snippet)

Create `.github/workflows/deploy.yml` with a simple pipeline:

```yaml
name: CI/CD - Build & Push to ECR
on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
      - name: Login to ECR
        run: |
          aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | docker login --username AWS --password-stdin ${{ secrets.AWS_ECR_LOGIN_URI }}
      - name: Build, Tag, Push
        run: |
          docker build -t ${{ secrets.ECR_REPOSITORY_NAME }}:latest .
          docker tag ${{ secrets.ECR_REPOSITORY_NAME }}:latest ${{ secrets.AWS_ECR_LOGIN_URI }}/${{ secrets.ECR_REPOSITORY_NAME }}:latest
          docker push ${{ secrets.AWS_ECR_LOGIN_URI }}/${{ secrets.ECR_REPOSITORY_NAME }}:latest
```

> For full deploy automation add a second job (or runner) that SSHs to EC2 / triggers webhook to pull the new image and restart the container.

---

## ✅ Tips & Troubleshooting

* If `app.py` shows `port in use`: change the port in the code or kill the process.
* If Docker not found on EC2: ensure the `usermod` and `newgrp` steps were applied and you reconnected the session.
* AWS ECR auth errors: confirm `AWS_REGION` and `AWS_ECR_LOGIN_URI` values in GitHub secrets.

---

## ✍️ Contributing

1. Fork the repo
2. Create a feature branch `git checkout -b feat/awesome-thing`
3. Make changes, add tests where applicable
4. Create a PR describing why this improves the project

---

## 📜 License

This project is available under the MIT License.

---

## 📬 Contact

Rayhan Hussain — `rayhanhcse` on GitHub

---

*Made with ♥ and balanced gradients.*
