pipeline {
    agent any

    environment {
        DOCKER_HOME = "/Applications/Docker.app/Contents/Resources/bin"
        DOCKER_BIN  = "${DOCKER_HOME}/docker"
        KUBECTL_BIN = "${DOCKER_HOME}/kubectl"

        DOCKER_REPO = "rajaditya079/swe645-assignment2-amd64"
        IMAGE_TAG   = "latest"
        FULL_IMAGE  = "${DOCKER_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    $DOCKER_BIN build -t $FULL_IMAGE .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "Logging into DockerHub..."
                        echo $DOCKER_PASS | $DOCKER_BIN login -u $DOCKER_USER --password-stdin

                        echo "Pushing image to DockerHub..."
                        $DOCKER_BIN push $FULL_IMAGE
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
                        echo "Applying Kubernetes deployment..."
                        $KUBECTL_BIN apply -f deployment.yaml

                        echo "Updating image to $FULL_IMAGE"
                        $KUBECTL_BIN set image deployment/swe645-deployment \
                        swe645-container=$FULL_IMAGE

                        echo "Forcing rollout restart..."
                        $KUBECTL_BIN rollout restart deployment/swe645-deployment

                        echo "Waiting for rollout..."
                        $KUBECTL_BIN rollout status deployment/swe645-deployment

                        echo "Deployment completed successfully."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful: ${FULL_IMAGE}"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}