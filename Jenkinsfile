pipeline {
    agent any

    environment {
        DOCKER_HOME = "/Applications/Docker.app/Contents/Resources/bin"
        DOCKER_BIN  = "${DOCKER_HOME}/docker"
        KUBECTL_BIN = "${DOCKER_HOME}/kubectl"

        DOCKER_REPO = "rajaditya079/swe645-webapp"
        IMAGE_TAG   = "latest"                     // always latest
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
                    docker buildx build \
                    --platform linux/amd64 \
                    --push \
                    -t $FULL_IMAGE .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                    $DOCKER_BIN push $FULL_IMAGE
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
                        # Apply deployment (creates if missing)
                        $KUBECTL_BIN apply -f deployment.yaml

                        # Update deployment with latest image
                        $KUBECTL_BIN set image deployment/swe645-deployment \
                        swe645-container=$FULL_IMAGE --record

                        # Wait for rollout to complete
                        $KUBECTL_BIN rollout status deployment/swe645-deployment

                        echo "Deleting old pods one by one while new pods are running..."
                        TIMEOUT=120
                        ELAPSED=0

                        while true; do
                            OLD_PODS=$($KUBECTL_BIN get pods -l app=swe645-app -o jsonpath='{.items[?(@.spec.containers[0].image!="'$FULL_IMAGE'")].metadata.name}')
                            if [ -z "$OLD_PODS" ]; then
                                echo "No old pods remaining."
                                break
                            fi
                            for pod in $OLD_PODS; do
                                echo "Deleting old pod: $pod"
                                $KUBECTL_BIN delete pod $pod --grace-period=5
                            done
                            sleep 5
                            ELAPSED=$((ELAPSED+5))
                            if [ $ELAPSED -ge $TIMEOUT ]; then
                                echo "Timeout reached: some old pods did not terminate within $TIMEOUT seconds."
                                exit 1
                            fi
                        done

                        echo "All old pods removed. Deployment fully clean."
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