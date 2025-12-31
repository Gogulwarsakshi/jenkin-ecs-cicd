pipeline {
    agent any

   environment {
    AWS_REGION  = "ap-southeast-2"
    ECS_CLUSTER = "react-cluster"
    ECS_SERVICE = "react-task-service-hizlx1m9"
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
          docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
          docker push ${ECR_URI}:${IMAGE_TAG}
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
        sh """
        aws ecs update-service \
          --cluster ${ECS_CLUSTER} \
          --service ${ECS_SERVICE} \
          --force-new-deployment \
          --region ${AWS_REGION}
        """
    }
}
