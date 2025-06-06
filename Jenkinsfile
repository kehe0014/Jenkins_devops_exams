pipeline {
    agent any

    environment {
        DOCKER_ID              = "tdksoft"
        DOCKER_IMAGE_MS        = "devops-eval-movie-service"
        DOCKER_IMAGE_CS        = "devops-eval-cast-service"
        DOCKER_TAG             = "v.${BUILD_ID}.0"
        ENVIRONMENTS           = 'dev qa staging prod'
        DOCKER_COMPOSE_FILE    = 'docker-compose.yml'
        DOCKER_COMPOSE_CMD     = 'docker compose'
        KUBECONFIG             = credentials("config")
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    echo "🔧 Building Docker images using Docker Compose..."
                    sh """
                    TAG=${DOCKER_TAG} ${DOCKER_COMPOSE_CMD} -f ${DOCKER_COMPOSE_FILE} build
                    sleep 6
                    """
                }
            }
        }

        stage('Push Docker Images') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    echo "📤 Pushing Docker images to Docker Hub..."
                        sh """
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE_MS:$DOCKER_TAG
                        docker push $DOCKER_ID/$DOCKER_IMAGE_CS:$DOCKER_TAG
                        """
                }
            }
        }

        stage('Create Kubernetes Namespaces') {
            steps {
                script {
                    echo "📂 Creating Kubernetes namespaces..."
                    sh """
                    for env in $ENVIRONMENTS; do
                        kubectl create namespace \$env --dry-run=client -o yaml | kubectl apply -f -
                    done
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    deployToEnvironment('dev')
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    deployToEnvironment('staging')
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input(message: 'Approve production deployment?', ok: 'Deploy to Prod')
                }
                script {
                    deployToEnvironment('prod')
                }
            }
        }
    }

    post {
        always {
            echo "🔚 Pipeline completed. Cleaning up..."
            sh 'docker system prune -f'
        }
        success {
            echo "✅ Pipeline succeeded!"
            // Add notification here (Slack, email, etc.)
        }
        failure {
            echo "❌ Pipeline failed!"
            // Add failure notification here
        }
    }
}

// Custom function for environment deployment
def deployToEnvironment(String env) {
    echo "🚀 Deploying to '${env}' environment..."
    withEnv(["DEPLOY_ENV=${env}"]) {
        sh """
        rm -rf .kube && mkdir -p .kube
        cat \$KUBECONFIG > .kube/config
        cp charts/values.yaml values.yml
        sed -i "s+tag.*+tag: ${env.DOCKER_TAG}+g" values.yml

        helm lint charts

        helm upgrade --install app-movie charts \\
            --values=values.yml \\
            --namespace ${env} \\
            --create-namespace \\
            --wait

        helm upgrade --install app-cast charts \\
            --values=values.yml \\
            --namespace ${env} \\
            --create-namespace \\
            --wait
        """
    }
}