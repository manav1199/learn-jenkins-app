pipeline {
    agent any

    stages {
        stage('W/O Docker') {
            steps {
                sh 'echo "Jenkins without docker"'
                sh 'touch without.txt'
            }
        }
        stage('With Docker')
        {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh 'echo "Jenkins with Docker"'
                sh 'npm --version'
                sh 'touch with.txt'
            }
        }
    }
}
