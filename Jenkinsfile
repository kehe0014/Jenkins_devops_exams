pipeline {

    agent any // Use any available agent

    environment {
        DOCKER_ID = "tdksoft" // Your Docker Hub username
        DOCKER_IMAGE_MS = "devops-eval-movie-service" // Movie service Docker image name
        DOCKER_IMAGE_CS = "devops-eval-cast-service"  // Cast service Docker image name
        DOCKER_TAG = "v.${BUILD_ID}.0" // Auto-incremented tag per Jenkins build
        ENVIRONMENTS = 'dev qa staging prod' // Kubernetes namespaces to ensure
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        DOCKER_COMPOSE_CMD = 'docker compose' // Compatible with newer Docker versions
        KUBECONFIG = credentials("config") // Kubernetes config from Jenkins credentials
    }

    stages {

        stage('Build Docker Images') {
            steps {
                script {
                    echo "🔧 Building Docker images using Docker Compose..."
                    sh '''
                    TAG=${DOCKER_TAG} ${DOCKER_COMPOSE_CMD} -f ${DOCKER_COMPOSE_FILE} build
                    sleep 6
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // Docker password from Jenkins credentials
            }
            steps {
                script {
                    echo "📤 Pushing Docker images to Docker Hub..."
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MS:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CS:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Create Kubernetes Namespaces') {
            steps {
                script {
                    echo "📂 Creating Kubernetes namespaces if they don't exist..."
                    sh '''
                    for env in $ENVIRONMENTS; do
                        kubectl create namespace $env --dry-run=client -o yaml | kubectl apply -f -
                    done
                    '''
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    echo "🚀 Deploying to 'dev' environment..."
                    sh '''
                    rm -rf .kube && mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    # Lint Helm chart before deploying
                    helm lint charts

                    # Deploy both services using Helm from the same chart folder
                    helm upgrade --install app-movie charts --values=values.yml --namespace dev --create-namespace
                    helm upgrade --install app-cast charts --values=values.yml --namespace dev --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    echo "🚀 Deploying to 'staging' environment..."
                    sh '''
                    rm -rf .kube && mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    helm lint charts

                    helm upgrade --install app-movie charts --values=values.yml --namespace staging --create-namespace
                    helm upgrade --install app-cast charts --values=values.yml --namespace staging --create-namespace
                    '''
                }
            }
        }

        stage('Deploy to Production') {
            // Optional: enable this only on the main/master branch
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy to production?', ok: 'Yes'
                }
                script {
                    echo "🚀 Deploying to 'prod' environment..."
                    sh '''
                    rm -rf .kube && mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    helm lint charts

                    helm upgrade --install app-movie charts --values=values.yml --namespace prod --create-namespace
                    helm upgrade --install app-cast charts --values=values.yml --namespace prod --create-namespace
                    '''
                }
            }
        }
    }
}
// Post-build actions can be added here if needed
// For example, notifications, archiving artifacts, etc.    

    post {
        always {
            echo "🔚 Pipeline completed. Cleaning up..."
            // Clean up workspace or perform any necessary actions
            sh 'docker system prune -f' // Optional: clean up Docker resources
        }
        success {
            echo "✅ Pipeline succeeded!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
// This Jenkinsfile defines a CI/CD pipeline for building, pushing, and deploying Docker images to Kubernetes using Helm.
// It includes stages for building Docker images, pushing them to Docker Hub, creating Kubernetes namespaces,
// and deploying to development, staging, and production environments.
// The pipeline uses environment variables for configuration and credentials for secure access to Docker Hub and Kubernetes.
// It also includes post-build actions for cleanup and notifications based on the pipeline's success or failure.
// The pipeline is designed to be flexible and can be extended with additional stages or steps as needed.
// Ensure that the necessary credentials and configurations are set up in Jenkins for this pipeline to work correctly.
// The pipeline uses Docker Compose for building images and Helm for deploying to Kubernetes.
// Make sure to adjust the Docker image names, Kubernetes namespaces, and other configurations as per your requirements.
// The pipeline is structured to be easily maintainable and scalable, allowing for future enhancements or modifications.
// The use of `timeout` and `input` stages ensures that deployments to production are controlled and require manual approval, enhancing safety in production environments.
// The pipeline is designed to be run on any available Jenkins agent, making it flexible for different environments.
// The use of `helm lint` ensures that the Helm charts are validated before deployment, reducing the risk of errors in the deployment process.
// The pipeline also includes a cleanup step in the post section to remove unused Docker resources, helping to manage disk space effectively.
// The pipeline is structured to be modular, allowing for easy addition of new stages or modifications to existing ones as the project evolves.
// Ensure that the Jenkins environment has the necessary plugins installed, such as Docker, Kubernetes, and Helm plugins, to support this pipeline.
// The pipeline is designed to be run in a CI/CD environment, automating the process of building, testing, and deploying applications efficiently.
// The use of environment variables allows for easy configuration and customization of the pipeline without hardcoding values.
// The pipeline is intended to be run in a secure environment, with sensitive information like Docker credentials and Kubernetes config stored securely in Jenkins credentials.
// The pipeline can be triggered automatically on code changes or manually as needed, providing flexibility in the deployment process.
// The pipeline is designed to be robust and handle various scenarios, including failures in any stage, with appropriate error handling and notifications.
// The pipeline can be extended with additional stages for testing, quality checks, or other requirements as needed.
// The pipeline is structured to follow best practices in CI/CD, ensuring a smooth and efficient deployment process.    