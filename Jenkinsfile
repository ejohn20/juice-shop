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
            sh './node_modules/.bin/dependency-check package.json app.js server.js'
            sh './node_modules/.bin/dependency-check --unused --ignore package.json app.js server.js'
          }
        }
        stage('Dependency Check') {
          steps {
            dependencyCheckAnalyzer datadir: 'dependency-check-data', isFailOnErrorDisabled: true, hintsFile: '', includeCsvReports: false, includeHtmlReports: false, includeJsonReports: false, isAutoupdateDisabled: false, outdir: '', scanpath: '', skipOnScmChange: false, skipOnUpstreamChange: false, suppressionFile: '', zipExtensions: ''
            dependencyCheckPublisher canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''
            archiveArtifacts allowEmptyArchive: true, artifacts: '**/dependency-check-report.xml', onlyIfSuccessful: true
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