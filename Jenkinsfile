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
          sh 'npm install --production --unsafe-perm -q -p > npm_install.log'
          archiveArtifacts "npm_install.log"
          //sh returnStdout: true, script: "grep 'WARN' ${env.outputDir}/npm_install_log > ${env.outputDir}/npm_install_warnings"
        }
        //input(message: 'Manual Security Review', id: 'sec1')
      }
    }
    stage('Test') {
      parallel {
        stage('Acceptance Tests') {
          agent {
            docker {
              image 'node:9.3'
              args "-p 4000:3000 -u root"
            }
          }
          steps {
            sh 'npm install --unsafe-perm'
            sh "npm test > npm_test.log"
            archiveArtifacts "npm_test.log"
          }
        }
        stage('ESLint Security') {
          agent {
            docker {
              image 'node:9.3'
              args "-p 4000:3000 -u root"
            }
          }
          steps {
            script{
              try {
                sh 'npm i -g --unsafe-perm eslint eslint-plugin-standard eslint-plugin-import eslint-config-standard eslint-plugin-security eslint-plugin-node'
                sh "eslint --no-eslintrc -c ./.eslintrc.json . eslint.log"
                archiveArtifacts "eslint.log"
                //sh 'npm install --unsafe-perm eslint-plugin-security'
                //sh "./node_modules/eslint/bin/eslint.js .*js > eslint-security.log"
                //archiveArtifacts "eslint-security.log"
              } catch(Exception e) {
                currentBuild.result = 'UNSTABLE'
              }
            }
          }
        }
        stage('Source Clear Dependency Check') {
          steps {
            script{
              try {
                withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                    sh "srcclr scan --json > srcclr.json"
                    archiveArtifacts "srcclr.json"
                }
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
            sh "docker run --network='host' -t owasp/zap2docker-stable zap-baseline.py -t http://127.0.0.1:3000 > zap.log"
            archiveArtifacts "zap.log"
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