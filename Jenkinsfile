pipeline {
  agent any
  environment {
    REACT_APP_VERSION = "1.0.$BUILD_ID"
    AWS_DEFAULT_REGION = "eu-central-1"
  }
  stages {
    stage('Deploy to AWS') {
      agent {
        docker {
          image 'amazon/aws-cli:2.34.0'
          args "-u root --entrypoint=''"
          reuseNode true
        }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-jenkins', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
          sh '''
          aws --version
          yum install -y jq
          LATEST_TD_REVISION = $(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq ".taskDefinition.revision")
          echo $LATEST_TD_REVISION
          aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
        '''
        }
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
  }
}