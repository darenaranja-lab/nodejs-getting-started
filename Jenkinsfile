pipeline {
    agent any
    environment {
        IMAGE_NAME = "node-app"
        CONTAINER_NAME = "node-app-container"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/darenaranja-lab/nodejs-getting-started.git', credentialsId: 'github-pat'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                '''
            }
        }

        stage('Run New Container') {
            steps {
                sh 'docker run -d -p 3000:3000 -e PORT=3000 --name $CONTAINER_NAME $IMAGE_NAME'
            }
        }
    }
}

