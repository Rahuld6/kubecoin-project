pipeline {
  agent any

  environment {
    DOCKER_USER = "rahuld06097"
    FRONTEND_IMAGE = "${DOCKER_USER}/kubecoin-frontend"
    BACKEND_IMAGE  = "${DOCKER_USER}/kubecoin-backend"
    ENV = ""
    TAG = ""
    KUBECONFIG = "/home/jenkins/.kube/config"
  }

  stages {

    stage('Identify Environment') {
      steps {
        script {
          if (env.BRANCH_NAME.contains('dev')) {
            env.ENV = 'dev'
            env.TAG = 'dev'
          } else if (env.BRANCH_NAME.contains('testing')) {
            env.ENV = 'testing'
            env.TAG = 'test'
          } else if (env.BRANCH_NAME.contains('production') || env.BRANCH_NAME.contains('main')) {
            env.ENV = 'production'
            env.TAG = 'prod'
          } else {
            error "Unknown branch: ${env.BRANCH_NAME}"
          }

          echo "Deploying to ${env.ENV} environment with tag ${env.TAG}"
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh """
          docker build --no-cache -t ${FRONTEND_IMAGE}:${TAG} frontend
          docker build --no-cache -t ${BACKEND_IMAGE}:${TAG} backend
        """
      }
    }

    stage('Push Images to DockerHub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-credentials',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh """
            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
            docker push ${FRONTEND_IMAGE}:${TAG}
            docker push ${BACKEND_IMAGE}:${TAG}
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh """
          kubectl get ns ${ENV} || kubectl create ns ${ENV}

          kubectl apply -f k8s/db.yaml -n ${ENV}
          kubectl apply -f k8s/backend.yaml -n ${ENV}
          kubectl apply -f k8s/frontend.yaml -n ${ENV}

          kubectl rollout status deployment/backend -n ${ENV}
          kubectl rollout status deployment/frontend -n ${ENV}
        """
      }
    }
  }
}
