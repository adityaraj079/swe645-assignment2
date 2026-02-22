pipeline {
    agent any

    environment {
        DOCKER_HOME = "/Applications/Docker.app/Contents/Resources/bin"
        DOCKER_BIN  = "${DOCKER_HOME}/docker"
        KUBECTL_BIN = "${DOCKER_HOME}/kubectl"

        DOCKER_REPO = "rajaditya079/swe645-assignment2-amd64"
        IMAGE_TAG   = "${BUILD_NUMBER}"
        FULL_IMAGE  = "${DOCKER_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push AMD64 Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "Setting up temporary Docker config..."
                        mkdir -p .docker
                        export DOCKER_CONFIG=$PWD/.docker

                        echo "Logging into DockerHub..."
                        echo $DOCKER_PASS | $DOCKER_BIN login -u $DOCKER_USER --password-stdin

                        echo "Creating buildx builder (if not exists)..."
                        $DOCKER_BIN buildx create --use --name amd64-builder || true

                        echo "Building and pushing linux/amd64 image..."
                        $DOCKER_BIN buildx build \
                            --platform linux/amd64 \
                            -t $FULL_IMAGE \
                            --push \
                            .

                        echo "Cleaning up Docker config..."
                        rm -rf .docker
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig-creds',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        echo "Updating deployment image to $FULL_IMAGE"

                        $KUBECTL_BIN set image deployment/swe645-deployment \
                        swe645-container=$FULL_IMAGE

                        echo "Waiting for rollout..."
                        $KUBECTL_BIN rollout status deployment/swe645-deployment

                        echo "Deployment successful."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment successful: ${FULL_IMAGE}"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}