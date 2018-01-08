pipeline {
  agent {
    dockerfile true
  }
  stages {
    stage('Build') {
      steps {
        dir /juice-shop {
          sh 'npm install eslint-plugin-security'
          sh 'npm install dependency-check'
          input(message: 'Manual Security Review', id: 'sec1')
        }
      }
    }
    stage('Test') {
      parallel {
        stage('App Tests') {
          steps {
            dir /juice-shop {
              sh 'npm test'
            }
          }
        }
        stage('ESLint Test') {
          steps {
            dir /juice-shop {
              sh './node_modules/eslint/bin/eslint.js server.js app.js package.json'
            }
          }
        }
        stage('NPM Dependency Check') {
          steps {
            dir /juice-shop {
              sh './node_modules/.bin/dependency-check package.json app.js server.js'
              sh './node_modules/.bin/dependency-check --unused --ignore package.json app.js server.js'
            }
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
        sh "clean"
      }
    }
  }
}