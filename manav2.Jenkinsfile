pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='bf33c59a-de5a-48c0-8235-4cc6dcf26141'
        NETLIFY_AUTH_TOKEN=credentials('netify-key')
        REACT_APP_VERSION="1.2.$BUILD_ID"
        AWS_ACCESS_KEY_ID=credentials('my-aws').username
        AWS_SECRET_ACCESS_KEY=credentials('my-aws').password
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
                //withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) 
                //{
                sh '''
                    aws --version
                    aws s3 ls
                    aws s3 sync build s3://$AWS_S3_BUCKET
                   '''
                //}
                }
        }
        stage('Test')
        {
            parallel{
                    stage('Unit Test')
                    {
                        agent{
                            docker{
                                image 'my-playwright'
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
                                image 'my-playwright'
                                reuseNode true
                            }
                        } 
                        steps{
                            sh '''
                                npm install serve
                                serve -s build &
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
        stage('Staging')
        {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                    }
                }
            environment{
                CI_ENVIRONMENT_URL="Will_be_Set"
                        } 
            steps{
                sh '''
                netlify --version
                echo "Deploying to production Site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --json > deploy-staging.json 
                CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-staging.json)               
                npx playwright test --reporter=html
                    '''
                }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                }
        }
        stage('Manual Approval')
        {
            steps{
                echo "This is the manual approval stage"
                timeout(time: 15,unit:'MINUTES')
                {
                    input message:'Provide approval to deploy to Prod',ok:'Deploy to Prod'
                }
            }
        }
        stage('Prod')
        {
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                    }
                }
            environment{
                CI_ENVIRONMENT_URL='https://manavwalunj.netlify.app'
                        } 
            steps{
                sh '''
                netlify --version
                echo "Deploying to production Site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod
                npx playwright test --reporter=html
                    '''
                }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                }
        }

    }

}
