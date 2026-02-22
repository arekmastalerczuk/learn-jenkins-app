pipeline {
    agent any

    stages {
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
        stage('Test build exists') {
          steps {
          sh '''
          echo "Running tests..."
          test -f build/index.html
          '''
          }
        }
        stage("Run existing tests") {
          agent {
            docker {
              image 'node:24-alpine'
              reuseNode true
            }
          }
          steps {
            sh '''
            echo "Run existing tests..."
            npm run test
            '''
          }
        }
    }
}
