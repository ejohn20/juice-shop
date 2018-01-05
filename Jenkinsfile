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
            sh './node_modules/.bin/dependency-check package.json app.js server.js'
            sh './node_modules/.bin/dependency-check --unused --ignore package.json app.js server.js'
          }
        }
        stage('Dependency Check Plugin') {
          steps {
            dependencyCheckAnalyzer(datadir: './dependency-check-data', hintsFile: './dependencycheck-base-hint.xml', includeCsvReports: true, includeHtmlReports: true, includeJsonReports: true, outdir: './', scanpath: './', skipOnScmChange: true, skipOnUpstreamChange: true, suppressionFile: './suppressed_issues.xml', zipExtensions: '.zip', includeVulnReports: true)
            dependencyCheckPublisher()
            archiveArtifacts(allowEmptyArchive: true, artifacts: '**/dependency-check-report.xml', onlyIfSuccessful: true)
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