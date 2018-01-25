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
        script{
          def outputDir = '${env.BUILD_ID}'
        }
        sh 'mkdir /var/build_output/build_${outputDir}'
        sh 'npm install -q --production --unsafe-perm > /var/build_output/build_${outputDir}/install_log 2>&1 | tee -a /var/build_output/build_${outputDir}/install_log'
        //input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      parallel {
        stage('App Tests') {
          agent {
            docker {
              image 'node:9.3'
              args '-p 4000:3000 -u root'
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
              args '-p 4000:3000 -u root'
            }
          }
          steps {
            sh 'npm install --unsafe-perm eslint-plugin-security'
            sh './node_modules/eslint/bin/eslint.js server.js app.js'
          }
        }
        stage('Source Clear Dependency Check') {
          steps {
            sh 'srcclr scan'
            sh 'srcclr scan --json > scrclr.json'
          }
        }
      }
    }
    stage('Acceptance') {
      steps {
        script{
          try {
            sh 'pm2 start app --name "Juice-Shop"'
            sh 'docker run --network="host" -t owasp/zap2docker-stable zap-baseline.py -t http://127.0.0.1:3000'
          } catch(Exception e) {
            currentBuild.result = 'UNSTABLE'
          } finally {
            sh 'pm2 stop Juice-Shop'
            sh 'pm2 delete Juice-Shop'
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script{
          try{
            sh 'docker stop JuiceShop'
            sh 'docker rm JuiceShop'
          } catch(Exception e) {
            sh 'echo "No previously deployed container."'
          } finally {
            sh 'docker build . -t sdlc_demo:juiceshop'
            sh 'docker run -d -p 8888:3000 --name JuiceShop sdlc_demo:juiceshop'
          }
        }
      }
    }
    stage('Cleanup') {
      steps {
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenSuccess: true, cleanWhenUnstable: true)
      }
    }
  }
}