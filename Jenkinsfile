pipeline {
    agent any

    environment {
        DOCKER_HOME = "/Applications/Docker.app/Contents/Resources/bin"
        DOCKER_BIN  = "${DOCKER_HOME}/docker"
        KUBECTL_BIN = "${DOCKER_HOME}/kubectl"

        DOCKER_REPO = "rajaditya079/swe645-assignment2-amd64"
        IMAGE_TAG   = "${BUILD_NUMBER}"
        FULL_IMAGE  = "${DOCKER_REPO}:${IMAGE_TAG}"

        DOCKER_DEFAULT_PLATFORM = "linux/amd64"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image (AMD64)') {
            steps {
                sh '''
                    echo "Building Docker image for linux/amd64..."
                    export DOCKER_DEFAULT_PLATFORM=linux/amd64
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
                        mkdir -p .docker
                        export DOCKER_CONFIG=$PWD/.docker

                        echo $DOCKER_PASS | $DOCKER_BIN login -u $DOCKER_USER --password-stdin

                        $DOCKER_BIN push $FULL_IMAGE

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
                        $KUBECTL_BIN set image deployment/swe645-deployment \
                        swe645-container=$FULL_IMAGE

                        $KUBECTL_BIN rollout status deployment/swe645-deployment
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