pipeline {
    agent {
        label 'build-server-agent'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t company-website:latest .
                '''
            }
        }

        stage('Verify Image') {
            steps {
                sh '''
                    echo "Verifying Docker image..."
                    docker images
                    docker image inspect company-website:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker tag company-website:latest \
                            "$DOCKER_USER/company-website:latest"

                        docker push "$DOCKER_USER/company-website:latest"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Server') {
            agent {
                label 'deploy-server-agent'
            }

            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credential',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "Logging in to Docker Hub..."

                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        echo "Pulling latest Docker image..."

                        docker pull "$DOCKER_USER/company-website:latest"

                        echo "Removing previous container..."

                        docker stop company-website || true
                        docker rm company-website || true

                        echo "Starting new container..."

                        docker run -d \
                            --name company-website \
                            -p 9090:80 \
                            "$DOCKER_USER/company-website:latest"

                        docker logout

                        echo "Deployment completed!"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
