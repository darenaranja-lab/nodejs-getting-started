pipeline {
    agent any

    environment {
        APP_NAME = "node-app"
        CONTAINER_NAME = "node-app-container"
        APP_PORT = "5006"  // Node app listens on 5006
        HOST_PORT = "443"  // Nginx exposes HTTPS
        SSL_DIR = "/etc/ssl/node-app"
    }

    stages {
        stage('Clone Repo') {
            steps {
                echo "Cloning open-source repo..."
                git branch: 'main', url: 'https://github.com/heroku/node-js-sample.git'
                sh 'ls -l' // Show workspace contents in console
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
                    docker rm -f $CONTAINER_NAME || true
                    docker ps -a
                """
            }
        }

        stage('Deploy Node App') {
            steps {
                echo "Running new Docker container for Node app..."
                sh """
                    docker run -d -p $APP_PORT:$APP_PORT --name $CONTAINER_NAME $APP_NAME
                    docker ps | grep $CONTAINER_NAME
                """
            }
        }

        stage('Setup Nginx & SSL') {
            steps {
                echo "Setting up Nginx reverse proxy with SSL..."
                sh """
                    # Create SSL directory
                    sudo mkdir -p $SSL_DIR

                    # Generate self-signed certificates
                    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
                        -keyout $SSL_DIR/nginx.key \
                        -out $SSL_DIR/nginx.crt \
                        -subj "/C=PH/ST=MetroManila/L=QuezonCity/O=DevOps/OU=Interview/CN=localhost"

                    # Write Nginx config
                    echo '
                    server {
                        listen 443 ssl;
                        server_name localhost;

                        ssl_certificate $SSL_DIR/nginx.crt;
                        ssl_certificate_key $SSL_DIR/nginx.key;

                        location / {
                            proxy_pass http://127.0.0.1:$APP_PORT;
                            proxy_set_header Host $host;
                            proxy_set_header X-Real-IP $remote_addr;
                            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                            proxy_set_header X-Forwarded-Proto $scheme;
                        }
                    }' | sudo tee /etc/nginx/sites-available/node-app.conf

                    # Enable site and reload Nginx
                    sudo ln -sf /etc/nginx/sites-available/node-app.conf /etc/nginx/sites-enabled/node-app.conf
                    sudo nginx -t
                    sudo systemctl restart nginx

                    # Show Nginx status
                    sudo systemctl status nginx | head -n 20
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Node app should be accessible via HTTPS (https://<VM-IP>)"
        }
    }
}

