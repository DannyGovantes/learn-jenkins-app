pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = "20dd6edb-c6c0-45e9-bc0f-35d8330090fb"
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
  }

  stages{
    stage('Build') {

      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          ls -la
          node --version
          npm --version

          npm ci
          npm run build
          ls -la
        '''
      }
    }

    stage('Run tests'){
      parallel{
          stage('Unit tests'){
            agent {
              docker {
                image 'node:18-alpine'
                reuseNode true
              }
            }
            steps{
              sh '''
                echo "Test stage"
                test -f build/index.html
                
                # [ -f "build/index.html" ] && echo "File exists" || echo "File does not exists"

                npm test
              '''
            }
            post {
              always {
                junit 'jest-results/junit.xml'
              }
            }
          
          }

        stage('E2E'){
            agent {
              docker {
                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                reuseNode true
              }
            }

            steps{

              sh'''
                  npm install serve
                  node_modules/.bin/serve -s build &
                  sleep 10
                  npx playwright test --reporter=html
              '''
            }
            post {
              always {
              
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local Report', reportTitles: '', useWrapperFileDirectly: true])
              }
            }
        }
     }
  }
      stage('Deploy staging'){

      agent {
        docker{
          image 'node:18-alpine'
          reuseNode true
        }
      }

      steps {

        sh'''
          npm install netlify-cli@20.1.1 node-jq
          node_modules/.bin/netlify --version
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy --dir ./build --json > deploy-output.json
        
        '''
        script{
          env.STAGE_URL= sh(script:"node_modules/.bin/node_jq -r '.deploy_url' deploy-output.json",returnStdout:truue)
        }
      }

      
    }
    stage('Staging E2E'){
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
          reuseNode true
        }
      }
      environment{
        
        CI_ENVIRONMENT_URL= "${env.STAGE_URL}"
      }

      steps{

        sh'''
            npx playwright test --reporter=html
        '''
      }
      post {
        always {
        
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'E2E STAGE Report', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
  }


    stage('Approval'){
      steps {
          timeout(time: 3, unit: 'MINUTES') {
            input message: 'Do you wish to deploy to production?', ok: 'Yes im sure'
        }
      }
    }
    stage('Deploy'){

      agent {
        docker{
          image 'node:18-alpine'
          reuseNode true
        }
      }

      steps {

        sh'''
          npm install netlify-cli@20.1.1
          node_modules/.bin/netlify --version
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy --dir ./build --prod
        '''
      }
    }
    stage('Prod E2E'){
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
          reuseNode true
        }
      }
      environment{
        
        CI_ENVIRONMENT_URL= "https://celebrated-peony-9ffb30.netlify.app"
      }

      steps{

        sh'''
            npx playwright test --reporter=html
        '''
      }
      post {
        always {
        
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'E2E Report', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
    }
  }
} 