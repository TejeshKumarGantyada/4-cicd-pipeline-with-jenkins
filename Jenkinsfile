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
