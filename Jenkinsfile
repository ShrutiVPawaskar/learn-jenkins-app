pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'b9cf0f39-7b7b-4c15-913a-8a2f4b4eda8c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        // Build stage using Node.js 18 Alpine image
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Building the application"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Run Tests') {
            parallel {
                stage('Test') {
                    // Test stage using the same Node.js 18 Alpine image
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'Test Stage'
                        sh'''
                            # test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml' 
                        }   
                    }
                }
                stage('E2E') {
                    // Test stage using the same Node.js 18 Alpine image
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.59.1-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'E2E Stage'
                        sh'''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml' 
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }   
                    }
                }
            }
        }
        stage('Deploy Staging'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to production Site ID: ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                '''    
                script{
                    env.STAGING_URL = sh(script: "jq -r '.deploy.ssl_url' deploy-output.json", returnStdout: true).trim()
                }  
            } 
        }
        stage('Staging E2E') {
            // Test stage using the same Node.js 18 Alpine image
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.59.1-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }  
            steps {
                echo 'E2E Prod Stage'
                sh'''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                }   
            }
        }
        stage('Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                     input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure.'
                }
            }
        }
        stage('Deploy Production'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production Site ID: ${NETLIFY_SITE_ID}"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --prod --dir=build --message="Automated deployment from Jenkins"
                '''
            }   
        }
        stage('Prod E2E') {
            // Test stage using the same Node.js 18 Alpine image
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.59.1-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "https://tourmaline-naiad-809390.netlify.app"
            }  
            steps {
                echo 'E2E Prod Stage'
                sh'''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }   
            }
        }
    }
}
