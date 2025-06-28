pipeline {
    agent any
    environment{
        REACT_APP_VERSION="1.2.$BUILD_ID"
    }

    stages {
        stage('Docker build')
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
        }
        stage('AWS DEPLOY')
        {
            agent{
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            environment{
                AWS_S3_BUCKET='manav-jenkins'
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                {
                sh '''
                    aws --version
                    aws s3 ls
                    aws s3 sync build s3://$AWS_S3_BUCKET
                   '''
                }
                }
        }
    }
}