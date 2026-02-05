pipeline {
    agent any

    environment {
        DOCKER_USER    = "rahuld06097"
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

                    if (env.BRANCH_NAME == 'dev') {
                        env.ENV = 'dev'
                        env.TAG = 'dev'
                    }
                    else if (env.BRANCH_NAME == 'testing') {
                        env.ENV = 'testing'
                        env.TAG = 'test'
                    }
                    else if (env.BRANCH_NAME == 'production' || env.BRANCH_NAME == 'main') {
                        env.ENV = 'production'
                        env.TAG = 'prod'
                    }
                    else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    echo "Deploying to namespace: ${env.ENV}"
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

        stage('Create Namespace (if not exists)') {
            steps {
                sh """
                  kubectl get namespace ${ENV} || kubectl create namespace ${ENV}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                  # Deploy Postgres resources first
                  kubectl apply -f k8s/level-4-namespaces-resources/postgres-pv.yaml -n ${ENV}
                  kubectl apply -f k8s/level-4-namespaces-resources/postgres-init-configmap.yaml -n ${ENV}
                  kubectl apply -f k8s/level-4-namespaces-resources/postgres-statefulset.yaml -n ${ENV}
                  kubectl apply -f k8s/level-4-namespaces-resources/postgres-headless-service.yaml -n ${ENV}

                  # Apply backend resources
                  kubectl apply -f k8s/level-4-namespaces-resources/backend-deployment.yaml -n ${ENV}
                  kubectl apply -f k8s/level-4-namespaces-resources/backend-service.yaml -n ${ENV}

                  # Apply frontend resources
                  kubectl apply -f k8s/level-4-namespaces-resources/frontend-deployment.yaml -n ${ENV}
                  kubectl apply -f k8s/level-4-namespaces-resources/frontend-service.yaml -n ${ENV}

                  # Apply secrets and configmaps
                  kubectl apply -f k8s/level-4-namespaces-resources/secret.yaml -n ${ENV}
                  kubectl apply -f k8s/level-4-namespaces-resources/configmap.yaml -n ${ENV}
                """
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully for ${env.BRANCH_NAME}"
        }
        failure {
            echo "❌ CI/CD Pipeline failed for ${env.BRANCH_NAME}"
        }
    }
}
