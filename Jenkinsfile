pipeline {
    agent any

    environment {
        DOCKER_USER   = "rahuld06097"
        FRONTEND_IMAGE = "rahuld06097/kubecoin-frontend"
        BACKEND_IMAGE  = "rahuld06097/kubecoin-backend"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Identify Environment') {
            steps {
                script {
                    echo "Branch detected: ${env.BRANCH_NAME}"

                    if (env.BRANCH_NAME.contains('dev')) {
                        env.ENV = 'dev'
                        env.TAG = 'dev'
                    }
                    else if (env.BRANCH_NAME.contains('testing')) {
                        env.ENV = 'testing'
                        env.TAG = 'test'
                    }
                    else if (env.BRANCH_NAME.contains('production') || env.BRANCH_NAME.contains('main')) {
                        env.ENV = 'production'
                        env.TAG = 'prod'
                    }
                    else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    echo "Environment: ${env.ENV}"
                    echo "Docker tag: ${env.TAG}"
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

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-credentials',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """
                      echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
                    """
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh """
                  docker push ${FRONTEND_IMAGE}:${TAG}
                  docker push ${BACKEND_IMAGE}:${TAG}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully for ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed for ${env.BRANCH_NAME}"
        }
    }
}
