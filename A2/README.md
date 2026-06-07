# Assignment II – Jenkins CI/CD Pipeline
## DSO101 – Continuous Integration and Continuous Deployment

**Student Name:** Kinley Pem  
**Student ID:** 02250354  
**Date of Submission:** 25th March

---

## Introduction

For this assignment, I had to set up a Jenkins pipeline to automate the build, test, and deployment of my To-Do List app from Assignment 1. The idea was to make Jenkins automatically pull the code from GitHub, install dependencies, run tests, build a Docker image, and push it to Docker Hub — without me having to do it manually each time.

My To-Do app has a React frontend, a Node.js/Express backend, and a PostgreSQL database. I ran Jenkins locally on my Windows laptop at `localhost:8080`.

---

## Tools and Technologies Used

| Tool | Purpose |
|------|---------|
| Jenkins 2.555.2 | CI/CD automation |
| GitHub | Hosting the source code |
| Node.js & npm | Running and building the backend |
| Jest & jest-junit | Running unit tests and generating reports |
| Docker & Docker Hub | Building and storing the Docker image |
| Visual Studio Code | Writing the Jenkinsfile |
| Windows | My operating system |

---

## How I Configured the Pipeline

### Task 1: Jenkins Setup

**Installing Jenkins**

I downloaded Jenkins 2.555.2 from jenkins.io and ran the installer on Windows. The setup wizard asked me to choose an install location — I kept it as the default `C:\Program Files\Jenkins\`. For the service logon, I selected "Run as LocalSystem". I set the port to `8080` and tested it to make sure it was free. For the Java path, it automatically detected my JDK at `C:\Program Files\Eclipse Adoptium\jdk-21.0.11.10-hotspot\`. I kept the "Start Service" option selected and finished the install.

**Unlocking Jenkins**

When I opened `localhost:8080` for the first time, it showed the Unlock Jenkins page. It told me to find the initial admin password at:

```
C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
```

I searched for that path using the Windows search bar, opened the file, copied the password, and pasted it into the browser. After that, it took me to the plugin installation page where I clicked **Install suggested plugins** and waited for everything to finish downloading.

**Creating Admin User**

Once plugins were done, I filled in the Create First Admin User form:

- Username: `kinleypem`
- Full Name: `Kinley Pem`
- Email: `02250354.cst@rub.edu.bt`

I kept the Jenkins URL as `http://localhost:8080/` and clicked finish.

**Installing the NodeJS Plugin**

I went to **Manage Jenkins > Plugins > Available plugins**, searched for `NodeJS`, checked it, and clicked Install. This was needed so Jenkins could run `npm` commands inside the pipeline.

**Configuring NodeJS in Tools**

I went to **Manage Jenkins > Tools** and added a NodeJS installation. I named it exactly `NodeJS` because that's what I referenced in the Jenkinsfile. I enabled automatic installation so Jenkins would download it on its own.

---

### Task 2: GitHub Repository Setup

My To-Do app was already on GitHub from Assignment 1. The repository is at:

```
https://github.com/kinleyp06/Kinley-Pem_02250354_DSO101_A1
```

The structure looks like this:

```
Kinley-Pem_02250354_DSO101_A1/
├── todo-app/
│   ├── backend/
│   └── frontend/
└── Jenkinsfile
```

**Generating a Personal Access Token**

I went to GitHub > Settings > Developer Settings > Personal Access Tokens and created a new token with `repo` and `admin:repo_hook` permissions. I copied the token right away since GitHub only shows it once.

**Adding Credentials to Jenkins**

I went to **Manage Jenkins > Credentials > Add Credentials** and entered:

- Kind: Username with password
- Username: `kinleyp06`
- Password: my GitHub PAT
- ID: `github-creds`

---

### Task 3: Writing the Jenkinsfile

I created a `Jenkinsfile` in the root of my repository. Since I'm on Windows, I used `bat` instead of `sh` for the shell commands. The pipeline has five stages:

```groovy
pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kinleyp06/Kinley-Pem_02250354_DSO101_A1.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('backend') {
                    bat 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('backend') {
                    bat 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t kinleyp06/be-todo:latest ./backend'
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push kinleyp06/be-todo:latest'
            }
        }

    }
}
```

**Setting up Jest for Testing**

The assignment required unit tests using Jest. I installed Jest and jest-junit inside the backend folder:

```bash
npm install --save-dev jest
npm install --save-dev jest-junit
```

Then I updated `package.json` to include the test script:

```json
{
  "scripts": {
    "test": "jest --ci --reporters=default --reporters=jest-junit"
  }
}
```

---

### Task 4: Running the Pipeline

I created a new Pipeline job in Jenkins:

1. Clicked **New Item**, gave it the name `todo-app-pipeline`, selected **Pipeline**, and clicked OK
2. Scrolled down to the Pipeline section and set:
   - Definition: **Pipeline script from SCM**
   - SCM: **Git**
   - Repository URL: `https://github.com/kinleyp06/Kinley-Pem_02250354_DSO101_A1.git`
   - Credentials: **github-creds**
   - Branch Specifier: `*/main`
   - Script Path: `Jenkinsfile`
3. Clicked **Save** then **Build Now**

---

## Pipeline Results

After fixing a few issues (explained below), the pipeline ran successfully through all five stages:

- **Checkout** – pulled the latest code from the main branch
- **Install Dependencies** – `npm install` ran inside the backend folder
- **Run Tests** – Jest ran the tests and generated a JUnit report
- **Build Docker Image** – built `kinleyp06/be-todo:latest` from the backend Dockerfile
- **Push Docker Image** – pushed the image to Docker Hub

### Links

| | Link |
|-|------|
| GitHub Repo | https://github.com/kinleyp06/Kinley-Pem_02250354_DSO101_A1 |
| Docker Hub Image | https://hub.docker.com/r/kinleyp06/be-todo |

---

## Challenges Faced

Honestly, this assignment had quite a few problems along the way.

**NodeJS name mismatch** – My Jenkinsfile said `nodejs 'NodeJS'` but I had named the tool something slightly different in Jenkins Tools. The pipeline kept failing at the start. I fixed it by making sure the name in Tools matched exactly.

**Jenkinsfile not found** – The first time I ran the pipeline, Jenkins couldn't find the Jenkinsfile. I forgot to push it to GitHub. After committing and pushing it to the repo root, it worked.

**GitHub authentication failing** – Jenkins couldn't connect to my repo at first. I had put in the wrong URL and hadn't properly added my PAT as a credential. Once I set up the credentials correctly under Manage Jenkins and linked them in the pipeline config, it connected.

**Wrong directory for npm commands** – My backend is inside `todo-app/backend`, not at the root level. Running `npm install` was failing because it was looking in the wrong folder. I wrapped the commands in `dir('backend')` in the Jenkinsfile to fix the path.

**No test script in package.json** – The Run Tests stage failed with "missing script: test" because I hadn't set up Jest yet. I installed Jest and jest-junit and added the test script to `package.json`.

**Docker wasn't running** – When Jenkins got to the Build Docker Image stage, it threw a Docker daemon connection error. I had forgotten to start Docker Desktop. Starting it before running the pipeline fixed the issue.

---

## What I Learned

- How to install and set up Jenkins from scratch on Windows
- How to write a Jenkinsfile using Declarative Pipeline syntax
- How to connect Jenkins to a GitHub repo using a Personal Access Token
- Why tool names in the Jenkinsfile have to match exactly what's configured in Jenkins Tools
- How to set up Jest for automated testing inside a CI pipeline
- How Docker integrates with a Jenkins pipeline to build and push images
- That most pipeline failures are configuration issues, not code issues — and reading the console output carefully is the best way to debug them

---

## Conclusion

Setting up the Jenkins pipeline was more involved than I expected, mostly because of all the small configuration details that had to be exactly right. But once everything was connected — GitHub, Jenkins, NodeJS, Docker — watching the pipeline go through all five stages automatically was pretty satisfying. It really shows how CI/CD removes the manual steps from deployment and makes the whole process more reliable.