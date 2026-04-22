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
                }
            }
        }
        stage('Deploy'){
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
    }
    post {
        always {
            junit 'jest-results/junit.xml' 
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }   
    }
    
}
