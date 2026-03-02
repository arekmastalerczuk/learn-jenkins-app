pipeline {
  agent any
  environment {
    NETLIFY_SITE_ID = 'f22ccac0-4f4e-450f-bb7f-acad81774893'
    NETLIFY_AUTH_TOKEN = credentials('netlify-jenkins-token')
    REACT_APP_VERSION = "1.0.$BUILD_ID"
  }
  stages {
    stage('Docker') {
      steps {
        sh 'docker build -t my-playwright .'
      }
    }
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
    stage('Tests') {
      parallel {
        stage('Unit tests') {
          agent {
            docker {
              image 'node:24-alpine'
              reuseNode true
            }
          }
          steps {
            sh '''
          echo "Test for exists build/index.html ..."
          test -f build/index.html
          echo "Status code for exists build/index.html: $?"
          '''
          }
          post {
            always {
              junit 'jest-results/junit.xml'
            }
          }
        }
        stage('E2E') {
          agent {
            docker {
              image 'my-playwright' // Custom Playwright docker image from Dockerfile
              reuseNode true
            }
          }
          steps {
            sh '''
          echo "E2E tests..."
          # added '&' at the end of serve command to run it in the background
          serve -s build &
          # sleep 10 seconds to wait for the server to start
          sleep 10
          echo "Run E2E tests..."
          npx playwright test --reporter=html
          '''
          }
          post {
            always {
              publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
            }
          }
        }
      }
    }
    stage('Deploy staging & E2E') {
      agent {
        docker {
          image 'my-playwright' // Custom Playwright docker image from Dockerfile
          reuseNode true
        }
      }
      environment {
        CI_ENVIRONMENT_URL="TO_BE_SET"
      }
      steps {
        sh '''
          netlify --version
          echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --no-build --json > deploy-output.json
          # CI_ENVIRONMENT_URL sets below is only visible inside lines in that script, also need to ser environment default value
          CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
          echo "Staging E2E tests..."
          npx playwright test --reporter=html
          '''
      }
      post {
        always {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
    }
    stage('Deploy production & E2E') {
      agent {
        docker {
          image 'my-playwright' // Custom Playwright docker image from Dockerfile
          reuseNode true
        }
      }
      environment {
        CI_ENVIRONMENT_URL = 'https://jenkins-ci-cd-course.netlify.app'
      }
      steps {
        sh '''
          node -v
          netlify --version
          echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
          netlify status
          netlify deploy --dir=build --prod --no-build
          echo "Production E2E tests..."
          npx playwright test --reporter=html
          '''
      }
      post {
        always {
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Production E2E', reportTitles: '', useWrapperFileDirectly: true])
        }
      }
    }
  }
}