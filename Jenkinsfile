pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t node-app .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker rm -f node-app-container || true
                docker run -d -p 3000:5006 --name node-app-container node-app
                '''
            }
        }
    }
}

