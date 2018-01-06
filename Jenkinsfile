pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
        sh 'npm install eslint-plugin-security'
        sh 'npm install dependency-check'
        input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      parallel {
        stage('App Tests') {
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
            sh 'makedir dependency-check-output'
            sh './node_modules/.bin/dependency-check --enableRetired --disableBundleAudit -f ALL -o ./dependency-check-output --project="JuiceShop" package.json app.js server.js'
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