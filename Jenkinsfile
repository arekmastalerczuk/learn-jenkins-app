pipeline {
  agent any
  environment {
    REACT_APP_VERSION = "1.0.$BUILD_ID"
    AWS_DEFAULT_REGION = "eu-central-1"
    AWS_ECS_CLUSTER = "LearnJenkinsApp-Cluster-Prod"
    AWS_ECS_SERVICE = "LearnJenkinsApp-Service-Prod"
    AWS_ECS_TD_PROD = "LearnJenkinsApp-TaskDefinition-Prod"
  }
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
    stage('Build docker image') {
      agent {
        docker {
          image 'amazon/aws-cli:2.34.0'
          args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
          reuseNode true
        }
      }
      steps {
        sh '''
          amazon-linux-extras install docker
          docker build -t my-learn-jenkins-app .
        '''
      }
    }
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
          LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq ".taskDefinition.revision")
          echo $LATEST_TD_REVISION
          aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
          aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
        '''
        }
      }
    }
  }
}