pipeline {

    agent any // we will use any agent to run our pipeline

    environment { // Declaration of environment variables
        DOCKER_ID = "tdksoft" // replace this with your docker-id
        DOCKER_IMAGE = "devopsexam" // replace this with your docker image name
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build

        ENVIRONMENTS = 'dev qa staging prod' // we will use this variable to create the namespaces in kubernetes
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        DOCKER_COMPOSE_CMD = 'docker compose' // Use 'docker compose' for newer Docker versions
        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
        // K8S_NAMESPACE = "${env.BRANCH_NAME == 'main' ? 'prod' : env.BRANCH_NAME}" // This line remains commented as it's a different logic
    }

    stages {

        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building Docker images from compose file..."
                    sh "${DOCKER_COMPOSE_CMD} -f ${DOCKER_COMPOSE_FILE} build"
                }
            }
        }
        stage('Push Docker Images') {
            steps {
                script {
                    echo "Pushing Docker images to Docker Hub..."
                    sh "${DOCKER_COMPOSE_CMD} -f ${DOCKER_COMPOSE_FILE} push"
                }
            }
        }
        stage('Deploy to Dev') {
            when {
                // This stage will only run if the current branch is 'develop'
                branch 'develop'
            }
            steps {
                script {
                    echo "Deploying to 'dev' environment..."
                    // Add your deployment steps for the 'dev' environment here.
                    // This could involve:
                    // - `kubectl apply -f k8s/dev/`
                    // - `helm upgrade --install dev-app ./helm-charts --namespace dev`
                    // - `docker push ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}` (if deploying directly)
                    echo "Deployment to dev environment triggered by develop branch."
                }
            }
        }
      
    }
}

