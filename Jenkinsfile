pipeline {
  agent {
    docker {
      image 'node:9.3-alpine'
      args '-p 3000:3000 -u root'
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
        sh 'npm install --unsafe-perm'
        sh 'npm install --unsafe-perm eslint-plugin-security'
        sh 'npm install --unsafe-perm dependency-check'
        input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      stage('App Tests') {
        steps {
          sh 'npm test'
        }
      }
      stage('ESLint Test') {
        steps {
          sh './node_modules/eslint/bin/eslint.js server.js app.js'
        }
      }
      stage('NPM Dependency Check') {
        steps {
          sh './node_modules/.bin/dependency-check package.json app.js server.js'
          sh './node_modules/.bin/dependency-check --unused --ignore package.json app.js server.js'
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