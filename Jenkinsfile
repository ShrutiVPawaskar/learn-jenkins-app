pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}" // Example of using Jenkins build number as version
        AWS_DEFAULT_REGION = 'ap-south-1' // Set your AWS region
        AWS_ECS_CLUSTER = 'caring-squirrel-jenkins-prod' // Set your ECS cluster name
        AWS_ECS_SERVICE = 'LearnJenkinsApp-Service-Prod' // Set your ECS service name
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefination-Prod' // Set your ECS task definition name

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
        stage('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                sh '''
                    echo "Building Docker image"
                    docker build -t myjenkinsapp .
                '''
            }
        }
        stage('AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my_aws', 
                                passwordVariable: 'AWS_SECRET_ACCESS_KEY', 
                                usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version 
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-defination-prod.json | jq '.taskDefinition.revision')
                        echo "Latest Task Definition Revision: $LATEST_TD_REVISION"
                        aws ecs update-service --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION --cluster $AWS_ECS_CLUSTER
                        aws ecs wait services-stable --services $AWS_ECS_SERVICE --cluster $AWS_ECS_CLUSTER
                    '''
                }
            }
        }
    }
}
