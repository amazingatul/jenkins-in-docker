#!groovy

pipeline {
  agent any
  stages {
    stage('Docker Build and Run') {
      steps {
        script {
          // Build Docker image
          sh 'docker build -t backend .'

          // Stop and remove any container using port 3000
          sh '''
            existing_container=$(docker ps -q --filter "publish=3000")
            if [ ! -z "$existing_container" ]; then
              echo "Stopping container on port 3000..."
              docker stop $existing_container
              docker rm $existing_container
            fi
          '''

          // Run the new container
          sh 'docker run -d -p 3000:3000 backend'
        }
      }
    }
  }
}
