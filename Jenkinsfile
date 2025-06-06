pipeline {

    agent any // we will use any agent to run our pipeline

    environment { // Declaration of environment variables
        DOCKER_ID = "tdksoft" // replace this with your docker-id
        DOCKER_IMAGE_MS = "devops-eval-movie-service" // this is the name of our docker image for movie service
        DOCKER_IMAGE_CS = "devops-eval-cast-service"   // this is the name of our docker image for cast service
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
                    sh ''' 
                    TAG=${DOCKER_TAG} ${DOCKER_COMPOSE_CMD} -f ${DOCKER_COMPOSE_FILE} build
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker Push') { //we pass the built images to our docker hub account
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {
                script {
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
                    echo "Creating Kubernetes namespaces..."
                    sh '''
                    for env in $ENVIRONMENTS; do
                        kubectl create namespace $env --dry-run=client -o yaml | kubectl apply -f -
                    done
                    '''
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
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod
                    echo "Deployment to dev environment triggered by develop branch."
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                // This stage will only run if the current branch is 'staging'
                branch 'staging'
            }
            steps {
                script {
                    echo "Deploying to 'staging' environment..."
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod
                    echo "Deployment to staging environment triggered by staging branch."
                    '''
                }
            }
        }
        stage('Deploy to Production') {
            when {
                // This stage will only run if the current branch is 'main'
                branch 'main'
            }
            steps {
                // Create an Approval Button with a timeout of 15 minutes.
                // this requires a manual validation in order to deploy on production environment
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    echo "Deploying to 'prod' environment..."
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp fastapi/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install app fastapi --values=values.yml --namespace prod
                    echo "Deployment to production environment triggered by main branch."
                    '''
                }
            }
        }
                  
      
    }
}

