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
                    sh '''
                    if [ $(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                        echo "Stopping and removing old container..."
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    else
                        echo "No old container to remove."
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
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}

