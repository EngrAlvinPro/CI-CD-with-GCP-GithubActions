📜 README.md - CI/CD with GitHub Actions & Google Cloud Run
md
Copy
Edit
# 🚀 CI/CD Pipeline with GitHub Actions & Google Cloud Run

This project demonstrates a **CI/CD pipeline** using **GitHub Actions** to automatically build, test, and deploy a Flask web application to **Google Cloud Run**.

## 🎯 Features
- ✅ **Flask Web App** with HTML & CSS
- ✅ **Dockerized** using `Dockerfile`
- ✅ **GitHub Actions** for automated deployment
- ✅ **Google Cloud Run** for hosting
- ✅ **Secrets Management** using GitHub Secrets

---

## 🏗️ Project Structure

/ci-cd-project │── app.py │── Dockerfile │── requirements.txt │── .github/workflows/deploy.yaml # GitHub Actions CI/CD workflow │── templates/ │ └── index.html │── static/ │ ├── style.css │ ├── script.js (optional) │ └── images/ │ ├── logo.png (optional) │── README.md

yaml
Copy
Edit

---

## ⚙️ Prerequisites

Before starting, ensure you have:

1. A **Google Cloud Platform (GCP) account**.
2. **Google Cloud SDK** installed:  
   ```sh
   gcloud auth login
Git & Docker installed on your machine.

GitHub Repository connected to your project.

🛠️ Setup Google Cloud
1️⃣ Enable Required GCP Services
Run the following command to enable necessary APIs:

sh
Copy
Edit
gcloud services enable run.googleapis.com \
    cloudbuild.googleapis.com \
    artifactregistry.googleapis.com
2️⃣ Create an Artifact Registry
sh
Copy
Edit
gcloud artifacts repositories create my-app-repo \
    --repository-format=docker \
    --location=us-central1 \
    --description="Docker repository for my app"
3️⃣ Create & Assign a Service Account
sh
Copy
Edit
gcloud iam service-accounts create github-actions \
    --display-name="GitHub Actions Deployment SA"
Grant permissions:

sh
Copy
Edit
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:github-actions@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/run.admin"

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:github-actions@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.serviceAccountUser"
🚀 Set Up GitHub Actions Workflow
1️⃣ Store GCP Credentials in GitHub
Generate a JSON key for your service account:

sh
Copy
Edit
gcloud iam service-accounts keys create key.json \
    --iam-account=github-actions@YOUR_PROJECT_ID.iam.gserviceaccount.com
Go to GitHub Repository → Settings → Secrets and Variables → Actions → New Repository Secret

Name: GCP_SA_KEY

Value: Paste the contents of key.json

Also, add:

GCP_PROJECT_ID → Your GCP project ID

📜 GitHub Actions Workflow ( .github/workflows/deploy.yaml )
yaml
Copy
Edit
name: Deploy to Google Cloud Run

on:
  push:
    branches:
      - main  # Trigger deployment on push to main

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
        docker build -t gcr.io/${{ secrets.GCP_PROJECT_ID }}/my-app .
        docker push gcr.io/${{ secrets.GCP_PROJECT_ID }}/my-app

    - name: Deploy to Cloud Run
      run: |
        gcloud run deploy my-app \
          --image gcr.io/${{ secrets.GCP_PROJECT_ID }}/my-app \
          --platform managed \
          --region us-central1 \
          --allow-unauthenticated
📝 Flask Web App (app.py)
python
Copy
Edit
from flask import Flask, render_template

app = Flask(__name__, static_folder='static', template_folder='templates')

@app.route("/")
def home():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
🎨 Web UI (static/style.css)
css
Copy
Edit
body {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
    text-align: center;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'Poppins', sans-serif;
}
.container {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 10px;
    padding: 30px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}
h1 { color: #ffeb3b; }
.button {
    padding: 10px 20px;
    background: #ff9800;
    color: white;
    border-radius: 5px;
    text-decoration: none;
    transition: all 0.3s ease;
}
.button:hover { background: #f44336; transform: scale(1.05); }
🐳 Dockerfile
dockerfile
Copy
Edit
FROM python:3.9

WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8080
CMD ["gunicorn", "-b", "0.0.0.0:8080", "app:app"]
📌 Deployment Instructions
1️⃣ Push Code to GitHub
sh
Copy
Edit
git add .
git commit -m "Initial commit with CI/CD setup"
git push origin main
2️⃣ GitHub Actions Triggers Deployment
GitHub Actions automatically builds & deploys your app to Google Cloud Run.

3️⃣ View Your Live App
Run this command to get your app URL:

sh
Copy
Edit
gcloud run services describe my-app --platform managed --region us-central1 --format 'value(status.url)'
Example output:

arduino
Copy
Edit
https://my-app-xyz.run.app
Open in your browser 🎉

🚀 Deploy a New Version
Make changes and push them:

sh
Copy
Edit
git add .
git commit -m "Updated UI"
git push origin main
GitHub Actions will automatically deploy the new version!

📌 Troubleshooting
Permission Denied? Ensure your service account has the right roles (run.admin, iam.serviceAccountUser).

500 Internal Server Error? Make sure Flask is correctly serving templates and static files.

GitHub Actions Fails? Check your GitHub Secrets & Workflow logs.

✨ Conclusion
This project sets up a CI/CD pipeline for a Flask web app using:

🐙 GitHub Actions for automation

🐳 Docker for containerization

🌍 Google Cloud Run for hosting

Now, every push to main automatically builds & deploys your app! 🚀

📌 Author
👤 Alvin - DevOps Enthusiast 🚀

yaml
Copy
Edit

---

This README will make your repo look **professional & beginner-friendly**! Let me know if 
