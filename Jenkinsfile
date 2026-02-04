pipeline {
  agent any

  environment {
    DOCKER_USER = "rahuld06097"
    FRONTEND_IMAGE = "${DOCKER_USER}/kubecoin-frontend"
    BACKEND_IMAGE  = "${DOCKER_USER}/kubecoin-backend"
    ENV = ""
    TAG = ""
  }

  stages {

    stage('Identify Environment') {
      steps {
        script {
          if (env.BRANCH_NAME == 'dev') {
            env.ENV = 'dev'
            env.TAG = 'dev'
          } else if (env.BRANCH_NAME == 'testing') {
            env.ENV = 'testing'
            env.TAG = 'test'
          } else if (env.BRANCH_NAME == 'production') {
            env.ENV = 'production'
            env.TAG = 'production'
          }
          echo "Deploying to ${env.ENV} with tag ${env.TAG}"
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh """
          docker build --no-cache -t ${env.FRONTEND_IMAGE}:${env.TAG} frontend
          docker build --no-cache -t ${env.BACKEND_IMAGE}:${env.TAG} backend
        """
      }
    }

    stage('Push Images to DockerHub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh """
            echo \$PASS | docker login -u \$USER --password-stdin
            docker push ${env.FRONTEND_IMAGE}:${env.TAG}
            docker push ${env.BACKEND_IMAGE}:${env.TAG}
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh """
          export KUBECONFIG=/home/jenkins/kubeconfig
          kubectl get ns ${env.ENV} || kubectl create ns ${env.ENV}
          kubectl apply -f k8s/db.yaml -n ${env.ENV}
          kubectl apply -f k8s/backend.yaml -n ${env.ENV}
          kubectl apply -f k8s/frontend.yaml -n ${env.ENV}
        """
      }
    }
  }
}
