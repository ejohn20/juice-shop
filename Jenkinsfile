pipeline {
  agent {
    node {          
      label 'master'
      customWorkspace '/var/lib/jenkins/workspace/juice-shop'
      withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
        sh "echo ${SRCCLR_API_TOKEN}"
      }
    }
  }
  environment {
    CI = 'true'
    npm_config_cache = 'npm-cache'
    outputDir = "/var/lib/jenkins/build_output/build_${env.BUILD_ID}"
  }
  stages {
    stage('Build') {
      steps {
        script{
          sh "mkdir -p ${env.outputDir}"
          sh "npm install --production --unsafe-perm -q 2>&1 | tee ${env.outputDir}/npm_install_log"
          sh "grep 'WARN' ${env.outputDir}/npm_install_log | tee ${env.outputDir}/npm_install_warnings"
        }
        //input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      parallel {
        stage('App Tests') {
          agent {
            docker {
              image 'node:9.3'
              args "-p 4000:3000 -u root -v ${env.outputDir}:${env.outputDir}"
            }
          }
          steps {
            sh 'npm install --unsafe-perm'
            sh "npm test > ${env.outputDir}/npm_test_log"
          }
        }
        stage('ESLint Test') {
          agent {
            docker {
              image 'node:9.3'
              args "-p 4000:3000 -u root -v ${env.outputDir}:${env.outputDir}"
            }
          }
          steps {
            sh 'npm install --unsafe-perm eslint-plugin-security'
            sh "./node_modules/eslint/bin/eslint.js server.js app.js > ${env.outputDir}/eslint_log"
          }
        }
        stage('Source Clear Dependency Check') {
          steps {
            script{
              try {
                sh 'srcclr scan'
                sh "srcclr scan --json > ${env.outputDir}/srcclr.json"
              } catch(Exception e) {
                currentBuild.result = 'UNSTABLE'
              }
            } 
          }
        }
      }
    }
    stage('Acceptance') {
      steps {
        script{
          try {
            sh 'pm2 start app --name "Juice-Shop"'
            sh "docker run --network="host" -t owasp/zap2docker-stable zap-baseline.py -t http://127.0.0.1:3000 > ${env.outputDir}/ZAP_log"
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