pipeline{
    agent any
    stages{
        stage('Building Images')
        {
            steps{
                sh '''
                    docker build -f CI/Dockerfile-aws-cli -t my-aws-cli . --no-cache
                    docker build -f CI/Dockerfile-playwright -t  my-playwright . --no-cache
                   '''
            }
        }
    }
}