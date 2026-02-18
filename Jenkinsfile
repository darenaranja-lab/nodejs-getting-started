pipeline {
    agent any
    environment {
        APP_NAME = "node-app"
        APP_PORT = "5006"
    }

    stages {
        stage('Clone Repo') {
            steps {
                echo "Cloning open-source repo..."
                git branch: 'main', url: 'https://github.com/heroku/node-js-sample.git'
                sh 'ls -l'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker build -t $APP_NAME .
                    docker images | grep $APP_NAME
                """
            }
        }

        stage('Stop Old Container') {
            steps {
                echo "Stopping old container if it exists..."
                sh """
                    docker rm -f $APP_NAME-container || true
                    docker ps -a
                """
            }
        }

        stage('Deploy Node App') {
            steps {
                echo "Running new Docker container for Node app..."
                sh """
                    docker run -d -p $APP_PORT:$APP_PORT --name $APP_NAME-container $APP_NAME
                    docker ps | grep $APP_NAME-container
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully. Node app should be accessible via http://<VM-IP>:$APP_PORT"
        }
        failure {
            echo "Pipeline failed. Check the console logs for errors."
        }
    }
}

