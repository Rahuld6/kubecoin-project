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
          echo "Branch name detected: ${env.BRANCH_NAME}"

          if (env.BRANCH_NAME.contains('dev')) {
            env.ENV = 'dev'
            env.TAG = 'dev'
          } else if (env.BRANCH_NAME.contains('testing')) {
            env.ENV = 'testing'
            env.TAG = 'test'
          } else if (env.BRANCH_NAME.contains('production'_
