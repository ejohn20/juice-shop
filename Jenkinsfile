pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
        input(message: 'Manual Security Review', id: 'sec1')
      }
    }
  }
}