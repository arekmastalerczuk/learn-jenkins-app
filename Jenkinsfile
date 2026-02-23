pipeline {
    agent any

    stages {
      /*
        stage('Build') {
          agent {
            docker {
              image 'node:24-alpine'
              reuseNode true
            }
          }

            steps {
                sh '''
                ls -lah
                node --version
                npm --version
                npm ci
                npm run build
                ls -lah
                '''
            }
        }
        */

        stage('Test') {
          agent {
            docker {
              image 'node:24-alpine'
              reuseNode true
            }
          }

          steps {
          sh '''
          echo "Test for exists build/index.html"
          test -f build/index.html
          echo $?
          echo "Run existing tests..."
          npm test
          '''
          }
        }
        
        stage('E2E') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.58.2-noble' // Playwright docker image
              reuseNode true
            }
          }

          steps {
          sh '''
          echo "E2E tests.."
          # npm ci
          npm run build
          npm i serve
          # added '&' at the end of serve command to run it in the background
          node_modules/.bin/serve -s build &
          # sleep 10 seconds to wait for the server to start
          sleep 10
          npx playwright test
          '''
          }
        }
    }

    post {
      always {
        junit 'jest-results/junit.xml'
      }
    }
}
