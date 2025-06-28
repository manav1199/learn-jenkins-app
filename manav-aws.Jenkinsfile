pipeline {
    agent any
    environment{
        REACT_APP_VERSION="1.2.$BUILD_ID"
        AWS_DEFAULT_REGION="ap-south-1"
    }

    stages {
        /*stage('Docker build')
        {
            steps{
                sh 'docker build -t my-playwright . --no-cache'
            }
        }
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
        }*/
        stage('AWS DEPLOY')
        {
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                {
                sh '''
                    aws --version
                    aws ecs register-task-definition --cli-input-json file://aws/task-definition.json
                    aws ecs update-service --cluster manav-fargate-2 --service LearnJenkinsApp-TaskDefinition-Prod-service-cuxq4ejp --task-definition LearnJenkinsApp-TaskDefinition-Prod:2
                   '''
                }
                }
        }
    }
}