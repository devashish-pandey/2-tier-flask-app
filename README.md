# 2-Tier Flask App — Jenkins CI/CD Learning Project

A hands-on project where I took a 2-tier Flask + MySQL application and built a full Jenkins CI/CD pipeline around it — from GitHub webhook triggers to automated Docker builds, Docker Hub pushes, and Compose-based deployment.

## 📦 Original Project
Forked from: [https://github.com/LondheShubham153/two-tier-flask-app]

## 🛠️ What I Added / Changed
- Jenkins Pipeline (`Jenkinsfile`) with staged CI/CD workflow
- GitHub Webhook integration for automatic pipeline triggers on push
- Pipeline configured from SCM (Jenkins pulls the pipeline definition directly from the repo)
- Automated Docker image build as part of the pipeline
- Automated Docker Hub push (tag + push)
- Docker Compose–based deployment (Flask + MySQL, with healthcheck)
- `.dockerignore` configuration to fix build context issues
- CI/CD troubleshooting — real pipeline failures and their fixes
- Documentation of errors and solutions (kept alongside the code, not just in my head)

---

## 🔄 CI/CD Workflow
```
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Build
   ↓
Test
   ↓
Docker Hub
   ↓
Docker Compose
   ↓
Flask + MySQL
```

**In words:** a `git push` to GitHub fires a webhook → Jenkins picks up the change and runs the pipeline → the app image is built → basic tests run → the image is pushed to Docker Hub → Docker Compose pulls/builds and redeploys the Flask + MySQL stack.

---

## ⚙️ Jenkins Pipeline Stages

| Stage | What Happens |
|---|---|
| **Code Checkout** | Jenkins pulls the latest code from the GitHub repo |
| **Docker Build** | Builds the Flask app image from the Dockerfile |
| **Test** | Runs basic tests/checks against the built image |
| **Push to Docker Hub** | Tags and pushes the image to Docker Hub |
| **Deploy** | Runs `docker compose up -d --build` to redeploy Flask + MySQL |

See [`Jenkinsfile`](./Jenkinsfile) for the full pipeline definition.

---

## 🔗 GitHub Webhook Setup
1. Jenkins job config → enable **"GitHub hook trigger for GITScm polling"**
2. GitHub repo → **Settings → Webhooks → Add webhook**
   - Payload URL: `http://<jenkins-server>:8080/github-webhook/`
   - Content type: `application/json`
   - Event: **Just the push event**

Once set up, every push to `main`/`master` automatically triggers a full pipeline run — no manual "Build Now" needed.

---

## 🐛 CI/CD Troubleshooting — Real Issues I Hit

### 1. Docker build failed — permission denied reading `mysql-data/`
```
checking context: no permission to read from
'/var/lib/jenkins/workspace/cicd-pra/mysql-data/auto.cnf'
```
**Cause:** My local MySQL data folder (`mysql-data/`, written by the running MySQL container) got swept into the Docker build context. Those files are owned by MySQL's internal container user, which the `jenkins` system user couldn't read.
**Fix:** Excluded it via `.dockerignore` and `.gitignore` — runtime database files should never be part of a build context.

### 2. Docker push failed — "repository name must be lowercase"
```
docker push ****/two-tier-flask-app:latest
repository name must be lowercase
```
**Cause:** The push stage referenced the **password** credential variable instead of the **username** variable when building the image path (Jenkins masking part of the log with `****` was the giveaway that a credential, not plain text, was in that string).
**Fix:** Corrected the variable reference (`$dockerHubUser`, not `$dockerHubPass`) and switched from `-p` to `--password-stdin` for safer credential handling.

📄 Full write-up with root-cause analysis for both issues: [`13-jenkins-debugging-real-issues.md`](../13-jenkins-debugging-real-issues.md)

---

## 🚀 How to Run This Locally
```bash
git clone <this-repo-url>
cd 2-tier-flask-app
docker compose up -d --build
curl http://localhost:5000/health
```

To trigger the full CI/CD flow instead of running locally, push a change to the connected GitHub repo and watch the pipeline run in Jenkins.

---

## 📁 Project Structure
```
2-tier-flask-app/
├── app.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── Jenkinsfile
├── .dockerignore
├── README.md
└── screenshots/
    ├── pipeline-success.png
    ├── github-webhook-config.png
    └── dockerhub-image.png
```

---

## 📌 Key Takeaway
Automating a deployment doesn't remove the need to understand what's happening underneath — every Jenkins failure I hit traced back to something I'd already learned manually with plain Docker commands (build context rules, credential handling, tagging conventions). CI/CD just means those same rules now apply inside someone else's execution environment, with less room to eyeball what went wrong — which is exactly why documenting each error mattered.

## 🔗 Related Notes
- [12 — Jenkins CI/CD Pipeline (concepts)](../12-jenkins-cicd-pipeline.md)
- [13 — Jenkins Debugging: Real Issues](../13-jenkins-debugging-real-issues.md)
- Full learning repo: https://github.com/devashish-pandey/docker-learning
