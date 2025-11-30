pipeline {
  agent any

  environment {
    DOCKER_HUB_REPO = 'vyshnavi525/my-app'
    IMAGE_TAG = "v${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build Image') {
      steps {
        bat 'docker build -t %DOCKER_HUB_REPO%:%IMAGE_TAG% .'
      }
    }

    stage('Login & Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat '''
          echo %PASS% | docker login -u %USER% --password-stdin
          docker tag %DOCKER_HUB_REPO%:%IMAGE_TAG% %DOCKER_HUB_REPO%:latest
          docker push %DOCKER_HUB_REPO%:%IMAGE_TAG%
          docker push %DOCKER_HUB_REPO%:latest
          docker logout
          '''
        }
      }
    }

    stage('Start Minikube') {
      steps {
        echo "Starting Minikube (if not already running)..."
        bat '''
        minikube status || minikube start
        '''
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        echo "Deploying application to Kubernetes..."
        bat '''
        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml
        kubectl get pods
        kubectl get svc
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Image pushed to Docker Hub: ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}"
      echo "✅ Deployed to Kubernetes via Minikube (deployment.yaml + service.yaml)."
    }
  }
}
