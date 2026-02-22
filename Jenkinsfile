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
          #echo "Test for exists build/index.html"
          #test -f build/index.html
          #echo $?
          echo "Run existing tests..."
          npm test
          '''
          }
        }
    }

    post {
      always {
        junit 'test-results/junit.xml'
      }
    }
}
