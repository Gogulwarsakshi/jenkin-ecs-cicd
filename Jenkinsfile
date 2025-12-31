pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "479929096401"
        ECR_REPO = "react-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CLUSTER_NAME = "react-cluster"
        SERVICE_NAME = "react-service"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:Gogulwarsakshi/jenkin-ecs-cicd.git'
            }
        }

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
                  docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                      aws ecr get-login-password --region $AWS_REGION | \
                      docker login --username AWS --password-stdin \
                      $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker tag $ECR_REPO:$IMAGE_TAG \
                  $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

                  docker push \
                  $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
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
                  --cluster $CLUSTER_NAME \
                  --service $SERVICE_NAME \
                  --force-new-deployment
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
