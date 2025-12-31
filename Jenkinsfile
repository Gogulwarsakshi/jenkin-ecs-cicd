pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build React') {
            steps {
                sh '''
                  npm install
                  npm run build
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t react-app .'
            }
        }
    }
}


