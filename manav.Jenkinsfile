pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='bf33c59a-de5a-48c0-8235-4cc6dcf26141'
        NETLIFY_AUTH_TOKEN=credentials('netify-key')
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
                   '''
            }
        }
        stage('Test')
        {
            parallel{
                    stage('Unit Test')
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
                        post{
                            always{
                                    junit 'jest-results/junit.xml'
                                  }
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
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }
                        post{
                            always{
                                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                                  }
                            }
                    }

            }
        }
        stage('Deploy')
        {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                   '''
            }
        }

    }

}
