# CI-CD-Pipeline-for-Node.js-App-using-GitHub-Actions

Here's the full content formatted for a **`README.md`** file on GitHub:

```markdown
# 🚀 CI/CD Pipeline for Node.js App using GitHub Actions

This project demonstrates how to set up a complete CI/CD pipeline for a Node.js app using GitHub Actions. It includes unit testing, Docker image building, DockerHub pushing, and deployment automation on every push to the `main` branch.

---

## ✅ Objective

Create a `.github/workflows/main.yml` to:

1. Run unit tests  
2. Build a Docker image  
3. Push it to DockerHub  
4. Trigger deployment on merge to the `main` branch  

---

## 🛠️ Prerequisites

1. **Node.js App** with a `Dockerfile` and `package.json`
2. DockerHub account
3. GitHub repository with code on `main` branch
4. GitHub Secrets configured:
   - `DOCKERHUB_USERNAME`
   - `DOCKERHUB_TOKEN`
   - 'KUBE_CONFIG'

---

## 📁 Project Structure

```bash
nodejs-app/
├── app.js
├── package.json
├── test/
│   └── app.test.js
├── Dockerfile
└── .github/
    └── workflows/
        └── main.yml
```
## 📄  file-setup.py

```python

import os

# Base path
base_path = "/home/lilia/VIDEOS/nodejs-app"

# Files and their contents
files = {
    "app.js": """const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

module.exports = app;

if (require.main === module) {
  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
}
""",
    "deployment.yaml": """apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
  labels:
    app: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: laly9999/nodejs-app:latest
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
""",
    "package.json": """{
  "name": "nodejs-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "supertest": "^6.3.0"
  }
}
""",
    "test/app.test.js": """const request = require('supertest');
const app = require('../app');

describe('GET /', () => {
  it('should return Hello World!', async () => {
    const response = await request(app).get('/');
    expect(response.statusCode).toBe(200);
    expect(response.text).toBe('Hello World!');
  });
});
""",
    "Dockerfile": """FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["node", "app.js"]
""",
    ".github/workflows/main.yml": """name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
    - name: Install dependencies
      run: npm install
    - name: Run tests
      run: npm test

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    - name: Log in to DockerHub
      run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin
    - name: Build Docker image
      run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/nodejs-app:latest .
    - name: Push Docker image
      run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/nodejs-app:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
    - name: Set up Kubeconfig
      run: echo "${{ secrets.KUBE_CONFIG }}" | base64 --decode > $HOME/.kube/config
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'latest'
    - name: Update Kubernetes Deployment Image
      run: |
        kubectl set image deployment/nodejs-app nodejs-app=${{ secrets.DOCKERHUB_USERNAME }}/nodejs-app:latest
        kubectl rollout status deployment/nodejs-app
"""
}

# Create directories and write files
for relative_path, content in files.items():
    file_path = os.path.join(base_path, relative_path)
    os.makedirs(os.path.dirname(file_path), exist_ok=True)
    with open(file_path, "w") as file:
        file.write(content)

"✅ All files successfully created in /home/lilia/VIDEOS/nodejs-app/"



```


---

## 📄 1. Dockerfile

A file that defines how to build a Docker image for your Node.js app.  

✅ Why it matters:  
 - Packages your app and its dependencies into a container image.  
 - Ensures it runs the same everywhere — on your laptop, server, or Kubernetes.  
 - Acts as the base for automated CI/CD deployment.  

```Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["node", "app.js"]
```

---

## 🧪 2. Unit Test: `test/app.test.js`
A file that contains tests (e.g., using Jest) to validate that your app logic works.  
✅ Why it matters:  
- Catches bugs early in the development process.
- Automatically run during CI (GitHub Actions).
 - Ensures your code is correct before it's deployed.


```js
test('basic math', () => {
  expect(2 + 2).toBe(4);
});
```

---

```js
const request = require('supertest');
const app = require('../app');

describe('GET /', () => {
  it('should return Hello World!', async () => {
    const response = await request(app).get('/');
    expect(response.statusCode).toBe(200);
    expect(response.text).toBe('Hello World!');
  });
});

```


> Install `jest`:
```bash
npm install --save-dev jest supertest
```

> Add to `package.json`:
```json
"scripts": {
  "test": "jest"
}
```

## Run Your Tests Locally
```bash
npm test
```

---

## ⚙️ 3. GitHub Actions Workflow: `.github/workflows/main.yml`

A CI/CD configuration that defines all the automation steps for:
 - Testing
 - Building the Docker image
 - Pushing it to DockerHub
 - Deploying to Kubernetes
✅ Why it matters:
 - Automates the development workflow.
 - Ensures that every push to main runs the full pipeline.
 - Eliminates manual deployment steps.

```yaml
name: CI/CD Pipeline

#This workflow runs automatically whenever code is pushed to the main branch.
#It's the trigger for your CI/CD pipeline.
on:
  push:
    branches:
      - main


jobs:
  #Runs your first job named "test" on an Ubuntu virtual machine provided by GitHub Actions.
  test:
    runs-on: ubuntu-latest
  
    steps:
    #Fetches your repo's latest code so it can be used in this job.
    - name: Checkout code
      uses: actions/checkout@v3

    #Installs Node.js v18, so your project can run in a consistent environment.
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'

    #Installs all required Node.js packages listed in package.json.
    - name: Install dependencies
      run: npm install

    #Runs your test cases (like jest) to verify that your code works as expected.
    - name: Run tests
      run: npm test

  #Runs your second job named "build-and-push" on an Ubuntu virtual machine provided by GitHub Actions.
  #Runs only if the test job passes
  build-and-push:
    needs: test
    runs-on: ubuntu-latest

    steps:
    #Checkout Code Again: Required again because each job runs in a separate environment.
    - name: Checkout code
      uses: actions/checkout@v3

    #Logs in securely to DockerHub using your secrets set in GitHub. This allows pushing Docker images.
    - name: Log in to DockerHub
      run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

    #Builds a Docker image using your Dockerfile. Tags it with your DockerHub repo name and latest.
    - name: Build Docker image
      run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/nodejs-app:latest .

    #Uploads the image to DockerHub. This makes the image available for deployment.
    - name: Push Docker image
      run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/nodejs-app:latest

  #Runs only after a successful image push
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest

    steps:
    #Decodes and writes your base64 kubeconfig to ~/.kube/config for kubectl to use
    - name: Set up Kubeconfig
      run: |
        echo "${{ secrets.KUBE_CONFIG }}" | base64 --decode > $HOME/.kube/config

    #Installs kubectl in the GitHub runner
    - name: Set up kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'latest'

    - name: Update Kubernetes Deployment Image
      run: |
        #Updates the image in your Kubernetes Deployment with the new DockerHub image
        kubectl set image deployment/nodejs-app nodejs-app=${{ secrets.DOCKERHUB_USERNAME }}/nodejs-app:latest
        #Waits until the new pods are running and healthy
        kubectl rollout status deployment/nodejs-app

```

---

## 🧪 4. Test Docker Locally

Manually build and run the Docker image to verify it works before pushing to production.  

✅ Why it matters:
 - Helps catch issues early, like runtime errors or missing files.
 - Validates that the Dockerfile and app work together before automated builds.

```bash
# Build the image
docker build -t nodejs-app .

# Run the container
docker run -p 3000:3000 nodejs-app

# Visit http://localhost:3000 in your browser
```

---

## 🔐 5. Configure GitHub Secrets

Secure storage of sensitive values (like DockerHub credentials or kubeconfig) in GitHub repository settings.
✅ Why it matters:
 - Keeps your credentials out of source code.
 - Allows GitHub Actions to log in to DockerHub and Kubernetes securely.
 - Enables end-to-end CI/CD automation without exposing secrets.

In your repository:

- Go to **Settings** → **Secrets and variables** → **Actions**
- Add:
  - 'KUBE_CONFIG': base64-encoded kubeconfig
  - `DOCKERHUB_USERNAME`
  - `DOCKERHUB_TOKEN` (from [DockerHub Access Token](https://hub.docker.com/settings/security))

You can generate the KUBE_CONFIG like this:
```bash
cat ~/.kube/config | base64 -w 0
```


---

## 🎯 Final Outcome

Every time you push to `main`, this workflow will:
- ✅ Run tests
- ✅ Build your Docker image
- ✅ Push the image to DockerHub
- ✅ Trigger deployment (can be extended to Kubernetes, EC2, etc.)

---


```

