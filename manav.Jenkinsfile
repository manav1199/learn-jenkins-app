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
        stage('Test')
        {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            } 
            steps{
                sh '''
                   echo "Running the test stage"
                   ls -la 
                   test -f build/index.html
                   npm test
                   '''
            }
        }
        stage('E2E')
        {
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.53.1-jammy'
                    reuseNode true
                }
            } 
            steps{
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    npx playwright test --reporter=html
                   '''
            }
        }

    }
    post{
        always{
            junit 'jest-results/junit.xml'
        }
    }
}
