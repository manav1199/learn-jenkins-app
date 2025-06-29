pipeline {
    agent any
    environment{
        REACT_APP_VERSION="1.2.$BUILD_ID"
        AWS_DEFAULT_REGION="ap-south-1"
        AWS_ECS_CLUSTER="manav-fargate-2"
        AWS_ECS_SERVICE="LearnJenkinsApp-TaskDefinition-Prod-service-cuxq4ejp"
        AWS_ECS_TASK="LearnJenkinsApp-TaskDefinition-Prod"
    }

    stages {
        stage('Build')
        {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                   echo "Building inside docker container"
                   ls -la
                   npm ci
                   npm run build
                   docker build -t jenkins-app . --no-cache
                   '''
            }
        }
        stage('AWS DEPLOY')
        {
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                {
                sh '''
                    aws --version
                    yum install jq -y
                    LATEST_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK:$LATEST_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                   '''
                }
                }
        }
    }
}