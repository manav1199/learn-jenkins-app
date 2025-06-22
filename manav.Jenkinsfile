pipeline {
    agent any

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
                   '''
            }
        }
    }
}
