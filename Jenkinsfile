pipeline {
    agent any

    environment {
        CONTAINER_PORT = "3000"
        HOST_PORT = "3000"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    def buildResult = sh(
                        script: '''#!/bin/bash
                        set -eo pipefail
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_NAME}:latest . 2>&1 | tee failure.log
                        ''',
                        returnStatus: true
                    )

                    if (buildResult != 0) {
                        error "❌ Docker build failed! Check failure.log"
                    }
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    sh '''
                    echo "Tagging Docker image..."
                    docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${NEW_STAGE_TAG}
                    docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:prodv1
                    '''
                }
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                script {
                    sh '''
                    echo "Running Trivy security scan..."
                    if docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image \
                        --exit-code 0 --severity HIGH,CRITICAL ${IMAGE_NAME}:${NEW_STAGE_TAG}; then
                        echo "✅ Trivy scan completed!"
                    else
                        echo "⚠️ Trivy scan found vulnerabilities, but continuing pipeline..."
                    fi
                    '''
                }
            }
        }

        stage('Stop Existing Container') {
            steps {
                script {
                    sh '''
                    echo "Stopping existing container..."
                    CONTAINER_ID=$(docker ps -q --filter "publish=${HOST_PORT}")
                    if [ -n "$CONTAINER_ID" ]; then
                        docker stop "$CONTAINER_ID" || true
                        docker rm "$CONTAINER_ID" || true
                    else
                        echo "No container running on port ${HOST_PORT}"
                    fi
                    '''
                }
            }
        }

        stage('Run New Docker Container') {
            steps {
                script {
                    sh '''
                    echo "Starting new container with latest image..."
                    docker run -d \
                        -p ${HOST_PORT}:${CONTAINER_PORT} ${IMAGE_NAME}:prodv1
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                echo "📩 Sending deployment email..."
                emailext (
                    subject: "🚀 Pipeline Status: ${currentBuild.currentResult} (Build #${BUILD_NUMBER})",
                    body: """
                    <html>
                    <body>
                    <p><strong>Pipeline Status:</strong> ${currentBuild.currentResult}</p>
                    <p><strong>Build Number:</strong> ${BUILD_NUMBER}</p>
                    <p><strong>Check the <a href="${BUILD_URL}">console output</a>.</strong></p>
                    </body>
                    </html>
                    """,
                    to: "${EMAIL_RECIPIENTS}",
                    from: "development.aayanindia@gmail.com",
                    replyTo: "atul.rajput@aayaninfotech.com",
                    mimeType: 'text/html'
                )
            }
        }
    }
}
