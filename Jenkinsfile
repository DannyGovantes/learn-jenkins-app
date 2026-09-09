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
              
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
              }
            }
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
          node_modules/.bin/netlify --dir ./build --prod
        '''
      }
    }

  }

}