pipeline {
  agent {
    node {          
      label 'master'
      customWorkspace '/var/lib/jenkins/workspace/juice-shop'
    }
  }
  environment {
    CI = 'true'
    npm_config_cache = 'npm-cache'
  }
  stages {
    stage('Build') {
      steps {
        sh 'npm install --production --unsafe-perm'
        input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      parallel {
        stage('App Tests') {
          agent {
            docker {
              image 'node:9.3'
              args '-p 3000:3000 -u root'
            }
          }
          steps {
            sh 'npm install --unsafe-perm'
            sh 'npm test'
          }
        }
        stage('ESLint Test') {
          agent {
            docker {
              image 'node:9.3'
              args '-p 3000:3000 -u root'
            }
          }
          steps {
            sh 'npm install --unsafe-perm eslint-plugin-security'
            sh './node_modules/eslint/bin/eslint.js server.js app.js'
          }
        }
        stage('NPM Dependency Check') {
          agent {
            docker {
              image 'node:9.3'
              args '-p 3000:3000 -u root'
            }
          }
          steps {
            sh 'npm install --unsafe-perm dependency-check'
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
        sh 'docker build . -t sdlc_demo:juiceshop'
        sh 'docker run -d -p 3000:3000 sdlc_demo:juiceshop'
      }
    }
    stage('Cleanup') {
      steps {
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true)
      }
    }
  }
}