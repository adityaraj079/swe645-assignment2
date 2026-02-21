pipeline {
    agent any

    environment {
        DOCKER_HOME = "/Applications/Docker.app/Contents/Resources/bin"
        DOCKER_BIN  = "${DOCKER_HOME}/docker"
        KUBECTL_BIN = "${DOCKER_HOME}/kubectl"

        DOCKER_REPO = "rajaditya079/swe645-webapp"
        IMAGE_TAG   = "v${BUILD_NUMBER}"            // keep your versioning
        FULL_IMAGE  = "${DOCKER_REPO}:${IMAGE_TAG}"

        DOCKER_CONFIG = "${WORKSPACE}/.docker"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Docker Config') {
            steps {
                sh '''
                    mkdir -p ${DOCKER_CONFIG}
                    echo '{}' > ${DOCKER_CONFIG}/config.json
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | $DOCKER_BIN login \
                        -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    # Multi-arch build: amd64 + arm64
                    $DOCKER_BIN buildx build \
                        --platform linux/amd64,linux/arm64 \
                        -t $FULL_IMAGE \
                        --push .
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig-creds',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        # Apply deployment from repo
                        $KUBECTL_BIN apply -f deployment.yaml

                        # Update deployment with new image
                        $KUBECTL_BIN set image deployment/swe645-deployment \
                        swe645-container=$FULL_IMAGE

                        # Wait for rollout to complete
                        $KUBECTL_BIN rollout status deployment/swe645-deployment

                        # Delete old ReplicaSets with 0 ready pods to avoid CrashLoopBackOff
                        for rs in $($KUBECTL_BIN get rs -o jsonpath='{.items[?(@.status.readyReplicas==0)].metadata.name}'); do
                            $KUBECTL_BIN delete rs $rs
                        done
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