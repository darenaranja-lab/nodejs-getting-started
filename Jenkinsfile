pipeline {
    agent any

    environment {
        IMAGE_NAME = "node-app"
        CONTAINER_NAME = "node-app-container"
        DOCKER_PORT = "3000"
    }

    stages {
        stage('Checkout Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/darenaranja-lab/nodejs-getting-started.git', credentialsId: 'github-pat'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Stop Old Container') {
            steps {
                script {
                    sh """
                    if [ \$(docker ps -q -f name=${CONTAINER_NAME}) ]; then
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    fi
                    """
                }
            }
        }

        stage('Run New Container') {
            steps {
                script {
                    sh "docker run -d -p ${DOCKER_PORT}:3000 --name ${CONTAINER_NAME} ${IMAGE_NAME}"
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sh "curl -f http://localhost:${DOCKER_PORT} || exit 1"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for errors.'
        }
    }
}

