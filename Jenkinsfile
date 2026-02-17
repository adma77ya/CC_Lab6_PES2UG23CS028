pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend image..."

                docker rmi -f backend-app || true
                docker build -t backend-app ./backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Deploying backend containers..."

                docker network inspect app-network >/dev/null 2>&1 || \
                docker network create app-network

                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Deploying nginx load balancer..."

                docker rm -f nginx-lb || true

                # Start nginx first
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # Copy config from Jenkins workspace
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx config
                docker exec nginx-lb nginx -s reload
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Running containers:"
                docker ps
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully.'
        }
        failure {
            echo '❌ Pipeline failed. Check logs.'
        }
    }
}
