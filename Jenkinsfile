pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}" // Example of using Jenkins build number as version
        AWS_DEFAULT_REGION = 'ap-south-1' // Set your AWS region
    }
    stages {

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            steps{

                withCredentials([usernamePassword(credentialsId: 'my_aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version 
                        aws ecs register-task-definition --cli-input-json file://aws/task-defination-prod.json
                        aws ecs update-service --service LearnJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-TaskDefination-Prod:2 --cluster caring-squirrel-jenkins-prod
                    '''
                }
            }
        }

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

    }
}
