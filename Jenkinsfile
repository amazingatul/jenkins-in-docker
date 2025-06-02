pipeline {
    agent any

    environment {
        IMAGE_NAME = "spar-back"
        NEW_STAGE_TAG = "latest"
        CONTAINER_PORT = "3000"
        HOST_PORT = "3000"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def buildResult = sh(
                        script: '''
                            set -eo pipefail
                            echo "Building Docker image..."
                            docker build -t ${IMAGE_NAME}:${NEW_STAGE_TAG} .
                        ''',
                        returnStatus: true
                    )
                    if (buildResult != 0) {
                        error "❌ Docker build failed!"
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh '''
                        echo "Stopping existing container if any..."
                        CONTAINER_ID=$(docker ps -q --filter "ancestor=${IMAGE_NAME}:${NEW_STAGE_TAG}")
                        if [ -n "$CONTAINER_ID" ]; then
                            docker stop $CONTAINER_ID || true
                            docker rm $CONTAINER_ID || true
                        else
                            echo "No container running from this image."
                        fi

                        echo "Starting new container..."
                        docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --name ${IMAGE_NAME}_container ${IMAGE_NAME}:${NEW_STAGE_TAG}
                    '''
                }
            }
        }
    }
}
