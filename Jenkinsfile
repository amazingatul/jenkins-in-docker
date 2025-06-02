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
                            echo "Building Docker image ${IMAGE_NAME}:${NEW_STAGE_TAG}..."
                            docker build -t ${IMAGE_NAME}:${NEW_STAGE_TAG} .
                        ''',
                        returnStatus: true
                    )
                    if (buildResult != 0) {
                        error "❌ Docker build failed!"
                    } else {
                        echo "✅ Docker image built successfully."
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh '''
                        echo "Checking for existing containers..."
                        CONTAINER_ID=$(docker ps -q --filter "ancestor=${IMAGE_NAME}:${NEW_STAGE_TAG}")
                        if [ -n "$CONTAINER_ID" ]; then
                            echo "Stopping container ${CONTAINER_ID}..."
                            docker stop $CONTAINER_ID || echo "Failed to stop container $CONTAINER_ID or container already stopped."
                            echo "Removing container ${CONTAINER_ID}..."
                            docker rm $CONTAINER_ID || echo "Failed to remove container $CONTAINER_ID or container already removed."
                        else
                            echo "No container running from image ${IMAGE_NAME}:${NEW_STAGE_TAG}."
                        fi

                        echo "Starting new container..."
                        docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --name ${IMAGE_NAME}_container ${IMAGE_NAME}:${NEW_STAGE_TAG}
                        echo "Container started and mapped to port ${HOST_PORT}."
                    '''
                }
            }
        }
    }
}
