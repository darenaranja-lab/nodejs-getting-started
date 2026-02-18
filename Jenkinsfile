pipeline {
    agent any
    environment {
        APP_NAME = "node-app"
        APP_PORT = "5006"
        SSL_DIR = "/etc/ssl/node-app"
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

        stage('Setup Nginx & SSL') {
            steps {
                echo "Setting up Nginx reverse proxy with SSL..."
                
                // Create SSL directory
                sh "sudo mkdir -p $SSL_DIR"

                // Generate self-signed certificate
                sh """
                    sudo openssl req -x509 -nodes -days 365 \
                        -newkey rsa:2048 \
                        -keyout $SSL_DIR/nginx.key \
                        -out $SSL_DIR/nginx.crt \
                        -subj "/CN=localhost"
                """

                // Nginx config
                sh """
                    echo '
                    server {
                        listen 443 ssl;
                        server_name localhost;

                        ssl_certificate $SSL_DIR/nginx.crt;
                        ssl_certificate_key $SSL_DIR/nginx.key;

                        location / {
                            proxy_pass http://127.0.0.1:$APP_PORT;
                            proxy_set_header Host \$host;
                            proxy_set_header X-Real-IP \$remote_addr;
                            proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
                            proxy_set_header X-Forwarded-Proto \$scheme;
                        }
                    }
                    ' | sudo tee /etc/nginx/sites-available/node-app.conf
                """

                // Enable site and restart Nginx
                sh """
                    sudo ln -sf /etc/nginx/sites-available/node-app.conf /etc/nginx/sites-enabled/
                    sudo nginx -t
                    sudo systemctl restart nginx
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully. Node app should be accessible via HTTPS (https://<VM-IP>)"
        }
        failure {
            echo "Pipeline failed. Check the console logs for errors."
        }
    }
}

