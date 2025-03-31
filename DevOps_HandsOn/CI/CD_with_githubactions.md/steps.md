# CI/CD with GitHub Actions and Google Cloud

## Overview
This guide walks you through setting up a CI/CD pipeline using **GitHub Actions** to deploy a **Flask web app** to **Google Cloud Run**.

---

## Step 1: Setup Google Cloud
### 1.1. Create a Google Cloud Project
1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Click **Select a project** → **New Project**.
3. Name it (e.g., `github-actions-demo`) and click **Create**.

### 1.2. Enable Required APIs
Run the following commands in **Cloud Shell** or **Terminal**:
```sh
gcloud services enable cloudbuild.googleapis.com \
    run.googleapis.com \
    artifactregistry.googleapis.com \
    iam.googleapis.com
```

### 1.3. Create a Service Account
```sh
gcloud iam service-accounts create github-actions \
    --description="GitHub Actions Service Account" \
    --display-name="GitHub Actions"
```

Grant necessary permissions:
```sh
gcloud projects add-iam-policy-binding devops-handson-446013 \
    --member="serviceAccount:github-actions@devops-handson-446013.iam.gserviceaccount.com" \
    --role="roles/cloudbuild.builds.editor"

gcloud projects add-iam-policy-binding devops-handson-446013 \
    --member="serviceAccount:github-actions@devops-handson-446013.iam.gserviceaccount.com" \
    --role="roles/run.admin"

gcloud projects add-iam-policy-binding devops-handson-446013 \
    --member="serviceAccount:github-actions@devops-handson-446013.iam.gserviceaccount.com" \
    --role="roles/storage.admin"
```

### 1.4. Generate a JSON Key for GitHub Actions
```sh
gcloud iam service-accounts keys create key.json \
    --iam-account=github-actions@devops-handson-446013.iam.gserviceaccount.com
```
This will download a `key.json` file. Keep it safe!

---

## Step 2: Store Secrets in GitHub
1. Go to **GitHub repository** → **Settings** → **Secrets and variables** → **Actions**.
2. Click **New repository secret** and add:
   - **`GCP_devops-handson-446013`** → Your Google Cloud project ID
   - **`GCP_SA_KEY`** → Paste the content of `key.json`

---

## Step 3: Create a Simple Web App
### 3.1. Create `app.py`
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, CI/CD with GitHub Actions and Google Cloud!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

### 3.2. Create a `Dockerfile`
```dockerfile
FROM python:3.9

WORKDIR /app
COPY . /app

RUN pip install flask

CMD ["python", "app.py"]
```

---

## Step 4: Set Up GitHub Actions
### 4.1. Create `.github/workflows/deploy.yml`
Create a new file:
```
.github/workflows/deploy.yml
```
Add the following:
```yaml
name: Deploy to Google Cloud Run

on:
  push:
    branches:
      - main  # Trigger deployment when code is pushed to main branch

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Authenticate with Google Cloud
      uses: google-github-actions/auth@v1
      with:
        credentials_json: ${{ secrets.GCP_SA_KEY }}

    - name: Set up Google Cloud SDK
      uses: google-github-actions/setup-gcloud@v1

    - name: Build and push Docker image
      run: |
        gcloud auth configure-docker
        docker build -t gcr.io/${{ secrets.GCP_devops-handson-446013 }}/my-app .
        docker push gcr.io/${{ secrets.GCP_devops-handson-446013 }}/my-app

    - name: Deploy to Cloud Run
      run: |
        gcloud run deploy my-app \
          --image gcr.io/${{ secrets.GCP_devops-handson-446013 }}/my-app \
          --platform managed \
          --region us-central1 \
          --allow-unauthenticated
```

---

## Step 5: Deploy and Test
### 5.1. Push Code to GitHub
```sh
git add .
git commit -m "Initial commit"
git push origin main
```

### 5.2. Check GitHub Actions
- Go to **GitHub Repo** → **Actions Tab**.
- The pipeline should be running.
- Once completed, the web app will be deployed.

### 5.3. Get the App URL
```sh
gcloud run services describe my-app --platform managed --region us-central1 --format 'value(status.url)'
```
Copy the URL and open it in a browser 🎉.

---

## Conclusion
✅ **GitHub Actions** for CI/CD
✅ **Google Cloud Run** for deployment
✅ **Docker** for containerization

Now your pipeline is fully automated! 🚀
