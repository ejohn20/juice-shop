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
    //zapOutputDir = "/var/lib/jenkins/zap_output/build_${env.BUILD_ID}"
  }
  stages {
    stage('Build') {
      steps {
        script{
          sh 'npm install --production --unsafe-perm -q -p > npm_install.log'
          archiveArtifacts "npm_install.log"
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
                sh 'npm i -g --unsafe-perm eslint eslint-plugin-standard eslint-plugin-import eslint-config-standard eslint-plugin-security eslint-plugin-node eslint-plugin-promise'
                sh "eslint --no-eslintrc -c ./.eslintrc.json . | tee eslint.log"
              } catch(Exception e) {
                currentBuild.result = 'UNSTABLE'
              } finally {
                archiveArtifacts "eslint.log"
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
                }
              } catch(Exception e) {
                currentBuild.result = 'UNSTABLE'
              } finally {
                archiveArtifacts "srcclr.json"
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
            // sh "mkdir -p ${zapOutputDir}"
            sh "docker run -u root -v /var/lib/jenkins/workspace/juice-shop:/zap/wrk/:rw --network='host' -t owasp/zap2docker-stable zap-baseline.py -t http://127.0.0.1:3000 -r zap.html -x zap.xml | tee zap.log"
          } catch(Exception e) {
            currentBuild.result = 'UNSTABLE'
          } finally {
            sh 'pm2 stop Juice-Shop'
            sh 'pm2 delete Juice-Shop'
            archiveArtifacts "zap.log"
            archiveArtifacts "zap.html"
            archiveArtifacts "zap.xml"
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