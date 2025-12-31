pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "479929096401"

        IMAGE_NAME = "react-app"
        ECR_REPO  = "react-app"
        IMAGE_TAG = "${BUILD_NUMBER}"

        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        CLUSTER_NAME = "react-cluster"
        SERVICE_NAME = "react-service"
    }

    stages {

        stage('Build') {
            steps {
                sh '''
                  npm install
                  npm run build
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                  npm test -- --watch=false || true
                '''

            }
        }

        stage('Code Quality Scan') {
            steps {
                sh '''
                  npm audit || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                      aws ecr get-login-password --region ${AWS_REGION} | \
                      docker login --username AWS --password-stdin ${ECR_URI}
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                  ${ECR_URI}/${ECR_REPO}:${IMAGE_TAG}

                  docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                  ${ECR_URI}/${ECR_REPO}:latest

                  docker push ${ECR_URI}/${ECR_REPO}:${IMAGE_TAG}
                  docker push ${ECR_URI}/${ECR_REPO}:latest
                '''
            }
        }

        stage('Approval for Production') {
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                  aws ecs update-service \
                  --cluster ${CLUSTER_NAME} \
                  --service ${SERVICE_NAME} \
                  --force-new-deployment \
                  --region ${AWS_REGION}
                '''
            }
        }
    }
}
