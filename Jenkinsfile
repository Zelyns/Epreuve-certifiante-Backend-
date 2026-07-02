pipeline {
  agent any

  environment {
    NODE_IMAGE = 'node:20-alpine'
    DOCKER_IMAGE = 'taskcamel/tasklist-backend:latest'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Run unit tests') {
      steps {
        sh 'npm test'
      }
      post {
        always {
          junit 'reports/junit.xml'
        }
      }
    }

    stage('Generate coverage') {
      steps {
        sh 'npm run test:coverage'
      }
    }

    stage('Build application') {
      steps {
        sh 'npm run build'
      }
    }

    stage('Static analysis') {
      steps {
        sh 'npx sonarqube-scanner -Dsonar.projectKey=tasklist-backend -Dsonar.sources=src -Dsonar.exclusions=src/__tests__/** -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}'
      }
    }

    stage('Build Docker image') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE} .' 
      }
    }

    stage('Security scan') {
      steps {
        sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}'
      }
    }

    stage('Publish image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
          sh 'docker push ${DOCKER_IMAGE}'
        }
      }
    }
  }
}
