pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh '''npm install
npm install eslint-plugin-security
npm install dependency-check
npm install sonarlint'''
        input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      steps {
        sh 'echo "test"'
      }
    }
    stage('Acceptance') {
      steps {
        sh 'echo "Acceptance"'
      }
    }
    stage('Deploy') {
      steps {
        sh 'echo "Deploy"'
      }
    }
    stage('Cleanup') {
      steps {
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true)
      }
    }
  }
}