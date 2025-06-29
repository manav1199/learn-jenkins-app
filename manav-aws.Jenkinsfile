pipeline {
    agent any
    environment{
        REACT_APP_VERSION="1.2.$BUILD_ID"
        AWS_DEFAULT_REGION="ap-south-1"
        AWS_ECS_CLUSTER="manav-fargate-2"
        AWS_ECS_SERVICE="LearnJenkinsApp-TaskDefinition-Prod-service-cuxq4ejp"
        AWS_ECS_TASK="LearnJenkinsApp-TaskDefinition-Prod"
        APP_NAME="jenkins-app"
        AWS_DOCKER_REGISTRY="554510949427.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages {
        stage('Build')
        {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps{
                sh '''
                   echo "Building inside docker container"
                   ls -la
                   npm ci
                   npm run build
                   '''
            }
        }
        stage('Build Docker Image')
        {
            agent{
                docker{
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION . --no-cache
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }
        stage('AWS DEPLOY')
        {
            agent{
                docker{
                    image 'my-aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                {
                sh '''
                    aws --version
                    LATEST_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK:$LATEST_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                   '''
                }
                }
        }
    }
}