pipeline {
    agent any

    stages {
        stage('Checkout Repo') {
            steps {
                git url: 'https://github.com/darenaranja-lab/nodejs-getting-started.git', branch: 'main', credentialsId: 'github-pat'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t node-app .'
                }
            }
        }

        stage('Stop Old Container') {
            steps {
                script {
                    // Stop and remove container if it exists
                    sh '''
                    if [ $(docker ps -aq -f name=node-app-container) ]; then
                        docker rm -f node-app-container
                    fi
                    '''
                }
            }
        }

        stage('Run New Container') {
            steps {
                script {
                    sh 'docker run -d -p 3000:3000 --name node-app-container node-app'
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sh 'curl -f http://localhost:3000 || exit 1'
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}

