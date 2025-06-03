#!groovy

pipeline {
  agent any
  stages {
    stage('Docker Build and Run') {
      steps {
        sh 'docker build -t backend .'
        sh 'docker run -d -p 3000:3000 backend'
      }
    }
  }
}
