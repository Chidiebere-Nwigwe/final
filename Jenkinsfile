
pipeline {
    agent any

    environment{
        AWS_DOCKER_REGISTRY = '924917172028.dkr.ecr.us-east-2.amazonaws.com'
        // your ECR repository name
        APP_NAME = 'my_final_image'
        AWS_DEFAULT_REGION = 'us-east-2'
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:22.13.1-alpine'
                    // for the same docker image, reuse
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # list all files
                    ls -la
                    node --version
                    npm --version
                    # npm install
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
 
        stage('Test') {
            agent {
                docker {
                    image 'node:22.13.1-alpine'
                    reuseNode true
                }
            }
 
            steps {
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '--entrypoint=""'
                }
            }

                environment{
                    AWS_S3_BUCKET = 'final-20250414'
                }
    
           
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-final', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws s3 ls
                        # echo "Hello S3!" > index.html
                        # aws s3 cp index.html s3://final-20250414/index.html
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
            }
        }
        stage('Build My Docker Image'){

            agent{
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-final', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    
                    sh '''
                        amazon-linux-extras install docker
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:latest
                    '''
                }
            }
        }
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root --entrypoint=""'
                }
            }
           
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-final', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                {  
                    sh '''
                        aws --version
                        yum install jq -y
                       
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster my-final-project-20250414 --service my-final-app-Service-Prod  --task-definition Final-TaskDefinition-Prod:$LATEST_TD_REVISION
                    '''
                }
            }
        }
    }
}
}
 