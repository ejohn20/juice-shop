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
      parallel {
        stage('Test') {
          steps {
            sh 'npm test'
          }
        }
        stage('ESLint Test') {
          steps {
            sh 'echo "eslint"'
          }
        }
        stage('Dependency Check') {
          steps {
            sh './node_modules/.bin/dependency-check ./package.json'
            sh 'dependency-check ./package.json --unused'
          }
        }
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