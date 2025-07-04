pipeline{
    agent any
    environment{
        wl_msg='Starting Build'
    }
    stages{
        stage('Build')
        {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode 'true'
                }
            }
            steps{
                sh '''
                    echo $wl_msg
                    npm ci
                    npm run build
                   '''
            }
        }
        stage('Test')
        {
            parallel{
                stage('Unit-Test')
                {
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode 'true'
                        }
                    }
                    steps{
                        sh '''
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
                            reuseNode 'true'
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
    }
}