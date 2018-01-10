pipeline {
  agent {
    docker {
      image 'node:9.3'
      args '-p 3000:3000'
      customWorkspace '/var/lib/jenkins/workspace/juice-shop'
    }
  }
  environment {
    CI = 'true'
  }
  stages {
    stage('Build') {
      steps {
        pwd
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
            sh './node_modules/eslint/bin/eslint.js server.js app.js package.json'
          }
        }
        stage('NPM Dependency Check') {
          steps {
            sh './node_modules/.bin/dependency-check package.json app.js server.js'
            sh './node_modules/.bin/dependency-check --unused --ignore package.json app.js server.js'
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