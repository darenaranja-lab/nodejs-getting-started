pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "node-app"
        CONTAINER_NAME = "node-app-container"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Stop Old Container') {
            steps {
                script {
                    sh '''#!/bin/bash
# Get the container ID if it exists
CONTAINER_ID=$(docker ps -aq -f name=${CONTAINER_NAME})
if [ ! -z "$CONTAINER_ID" ]; then
    echo "Stopping and removing old container..."
    docker rm -f $CONTAINER_ID
else
    echo "No existing container found."
fi
'''
                }
            }
        }

        stage('Run New Container') {
            steps {
                script {
                    sh "docker run -d -p 3000:3000 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sh '''#!/bin/bash
# Wait for container to start
sleep 5
RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000)
if [ "$RESPONSE" -eq 200 ]; then
    echo "App is running and healthy!"
else
    echo "Health check failed! HTTP status: $RESPONSE"
    exit 1
fi
'''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}

