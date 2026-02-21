pipeline {
    agent any

    environment {
        DOCKER_HOME = "/Applications/Docker.app/Contents/Resources/bin"
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