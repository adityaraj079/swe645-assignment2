pipeline {
    agent any

    environment {
        DOCKER_BIN = "/Applications/Docker.app/Contents/Resources/bin/docker"
        DOCKER_REPO = "rajaditya079/swe645-webapp"
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "${DOCKER_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh """
                    ${DOCKER_BIN} build -t ${FULL_IMAGE} .
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo ${DOCKER_PASS} | ${DOCKER_BIN} login -u ${DOCKER_USER} --password-stdin
                        ${DOCKER_BIN} push ${FULL_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig-creds',
                    variable: 'KUBECONFIG'
                )]) {
                    sh """
                        kubectl set image deployment/swe645-deployment \
                        swe645-container=${FULL_IMAGE} --record

                        kubectl rollout status deployment/swe645-deployment
                    """
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
