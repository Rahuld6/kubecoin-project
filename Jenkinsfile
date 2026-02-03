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
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh """
        docker build -t ${FRONTEND_IMAGE}:${TAG} frontend
        docker build -t ${BACKEND_IMAGE}:${TAG} backend
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
          docker push ${FRONTEND_IMAGE}:${TAG}
          docker push ${BACKEND_IMAGE}:${TAG}
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh """
        kubectl apply -f k8s/db.yaml -n ${ENV}
        kubectl apply -f k8s/backend.yaml -n ${ENV}
        kubectl apply -f k8s/frontend.yaml -n ${ENV}
        """
      }
    }
  }
}

