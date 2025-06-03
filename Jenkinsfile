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
                        script: '''#!/bin/bash
set -eo pipefail
echo "Building Docker image..."
docker build -t ${IMAGE_NAME}:latest . | tee failure.log
''',
                        returnStatus: true
                    )

                    if (buildResult != 0) {
                        error "❌ Docker build failed! Check failure.log"
                    }
                }
            }
        }

        stage('Run New Docker Container') {
            steps {
                script {
                    sh '''#!/bin/bash
echo "Starting new container with latest image..."
docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}
'''
                }
            }
        }
    }
}
