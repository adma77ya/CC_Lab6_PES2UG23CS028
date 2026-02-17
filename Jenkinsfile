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

                # Create network only if it doesn't exist
                docker network inspect app-network >/dev/null 2>&1 || \
                docker network create app-network

                # Remove old containers safely
                docker rm -f backend1 backend2 || true

                # Start backend containers
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

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  -v $WORKSPACE/nginx/default.conf:/etc/nginx/conf.d/default.conf \
                  nginx
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
            echo '✅ Pipeline executed successfully. NGINX load balancer is running.'
        }

        failure {
            echo '❌ Pipeline failed. Check console logs.'
        }

        always {
            sh '''
            # cleanup dangling images
            docker image prune -f || true
            '''
        }
    }
}
