# 🚀 CI/CD Pipeline for Node.js with Jenkins & Docker (Windows + Linux)

In this deeply detailed tutorial, we'll build a complete CI/CD pipeline using Jenkins to automate the following:

* Create a simple Node.js app
* Write a Dockerfile to containerize the app
* Install and configure Jenkins on Windows and Linux
* Write a Jenkinsfile to automate testing and Docker deployment.
* Use Jenkins secrets to store Docker credentials

---

## 🧱 Step 1: Create Your Node.js Application

### 1. Create the project directory

```bash
mkdir ci-cd-node-app-jenkins
cd ci-cd-node-app-jenkins
```

### 2. Initialize Node.js project

```bash
npm init -y
```

This creates a `package.json` file with default values.

### 3. Create a simple app

Create `index.js`:

```js
const sum = require('./sum');

console.log("App started. Sum of 2 and 3 is:", sum(2, 3));
```

Create `sum.js`:

```js
function sum(a, b) {
  return a + b;
}

module.exports = sum;
```

### 4. Add a test using Jest

Install Jest:

```bash
npm install --save-dev jest
```

Update `package.json`:

```json
"scripts": {
  "test": "jest"
}
```

Create `sum.test.js`:

```js
const sum = require('./sum');

test('adds 2 + 3 to equal 5', () => {
  expect(sum(2, 3)).toBe(5);
});
```

### 5. Add .gitignore

```
node_modules
.env
```

---

## 🐳 Step 2: Add Dockerfile

Create `Dockerfile`:

```Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

To test locally:

```bash
docker build -t yourusername/jenkins-node-app .
docker run yourusername/jenkins-node-app
```

---

## 🔧 Step 3: Install Jenkins

### 🔷 On Windows:

1. Download Jenkins from [jenkins.io](https://www.jenkins.io/download/)
2. Run the installer
3. Install recommended plugins
4. Create first admin user
5. Setup "Git" and "Node.js" in **Global Tool Configuration**

> 📌 Also install Docker Desktop and make sure it works via command line (`docker --version`)

### 🐧 On Linux (Ubuntu):

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk git docker.io
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update && sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Then:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## 🔑 Step 4: Add DockerHub Secrets to Jenkins

Navigate to:

* **Manage Jenkins → Credentials → Global → Add Credentials**

Add two "Secret text":

* ID: `docker-username`, Value: your Docker username
* ID: `docker-password`, Value: your Docker password

These will be injected using Jenkins `credentials()` syntax.

---

## 📝 Step 5: Create Jenkinsfile

### ▶ Jenkinsfile for Windows

```groovy
pipeline {
  agent any

  tools {
    nodejs "nodejs20"
  }

  environment {
    DOCKERHUB_USERNAME = credentials('docker-username')
    DOCKERHUB_PASSWORD = credentials('docker-password')
  }

  stages {
    stage('Checkout Code') {
      steps {
        git 'https://github.com/TejeshKumarGantyada/4-cicd-pipeline-with-jenkins.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        bat 'npm test'
      }
    }

    stage('Docker Build') {
      steps {
        bat "docker build -t %DOCKERHUB_USERNAME%/jenkins-node-app:latest ."
      }
    }

    stage('Docker Login') {
      steps {
        bat "echo %DOCKERHUB_PASSWORD% | docker login -u %DOCKERHUB_USERNAME% --password-stdin"
      }
    }

    stage('Docker Push') {
      steps {
        bat "docker push %DOCKERHUB_USERNAME%/jenkins-node-app:latest"
      }
    }
  }
}
```

### ▶ Jenkinsfile for Linux

```groovy
pipeline {
  agent any

  tools {
    nodejs "nodejs20"
  }

  environment {
    DOCKERHUB_USERNAME = credentials('docker-username')
    DOCKERHUB_PASSWORD = credentials('docker-password')
  }

  stages {
    stage('Checkout Code') {
      steps {
        git 'https://github.com/TejeshKumarGantyada/4-cicd-pipeline-with-jenkins.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test'
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t $DOCKERHUB_USERNAME/jenkins-node-app:latest ."
      }
    }

    stage('Docker Login') {
      steps {
        sh "echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin"
      }
    }

    stage('Docker Push') {
      steps {
        sh "docker push $DOCKERHUB_USERNAME/jenkins-node-app:latest"
      }
    }
  }
}
```

---

## ✅ Step 6: Run the Pipeline

1. In Jenkins, create a **New Item → Pipeline**
2. Under **Pipeline → Definition**, select **Pipeline from SCM**
3. Set:

   * SCM: Git
   * Repository URL: `https://github.com/TejeshKumarGantyada/4-cicd-pipeline-with-jenkins.git`
   * Script Path: `Jenkinsfile`
4. Click **Save**, then **Build Now**

---

## 🧪 Validate the Image

Go to Docker Hub → Your Repo → Check for `jenkins-node-app:latest`
Run locally if needed:

```bash
docker pull yourusername/jenkins-node-app
```
