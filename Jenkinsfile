pipeline {
    agent any

   environment {
    AWS_REGION     = "ap-south-2"
    AWS_ACCOUNT_ID = "479929096401"
    IMAGE_NAME     = "react-app"
    ECR_URI        = "479929096401.dkr.ecr.ap-south-2.amazonaws.com/react-app"
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
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-creds']]) {

            sh '''
              aws ecr get-login-password --region ap-south-2 | \
              docker login --username AWS --password-stdin \
              479929096401.dkr.ecr.ap-south-2.amazonaws.com
            '''
        }
    }
}


        stage('Push Image to ECR') {
    steps {
        sh '''
          docker tag ${IMAGE_NAME}:latest ${ECR_URI}:latest
          docker push ${ECR_URI}:latest
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
