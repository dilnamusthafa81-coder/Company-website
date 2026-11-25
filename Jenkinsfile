
pipeline {
    agent none

    environment {
        DOCKERHUB_USERNAME = 'YOUR_DOCKERHUB_USERNAME'
        IMAGE_NAME = 'company-website'
        BUILD_SERVER = 'build-server'
        DEPLOY_SERVER_IP = 'YOUR_DEPLOYMENT_SERVER_PRIVATE_IP'

        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        SSH_CREDENTIALS = 'deployment-server-ssh'
    }

    stages {

        stage('Checkout Code') {
            agent {
                label 'build-server'
            }
            steps {
                checkout scm

                echo 'GitHub code downloaded successfully'
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'build-server'
            }
            steps {
                sh '''
                    docker build \
                    -t $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER \
                    -t $DOCKERHUB_USERNAME/$IMAGE_NAME:latest .
                '''

                echo 'Docker image built successfully'
            }
        }

        stage('Login to Docker Hub') {
            agent {
                label 'build-server'
            }
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            agent {
                label 'build-server'
            }
            steps {
                sh '''
                    docker push $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER
                    docker push $DOCKERHUB_USERNAME/$IMAGE_NAME:latest
                    docker logout
                '''

                echo 'Docker image pushed successfully'
            }
        }

        stage('Deploy Application') {
            agent {
                label 'built-in'
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: "${SSH_CREDENTIALS}",
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=accept-new \
                        "$SSH_USER@$DEPLOY_SERVER_IP" \
                        "docker pull $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER &&
                         docker stop company-website || true"
                    '''

                    sh '''
                        ssh -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=accept-new \
                        "$SSH_USER@$DEPLOY_SERVER_IP" \
                        "docker rm -f company-website || true"
                    '''

                    sh '''
                        ssh -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=accept-new \
                        "$SSH_USER@$DEPLOY_SERVER_IP" \
                        "docker run -d \
                         --name company-website \
                         --restart unless-stopped \
                         -p 9090:80 \
                         $DOCKERHUB_USERNAME/$IMAGE_NAME:$BUILD_NUMBER"
                    '''
                }

                echo 'Application deployed successfully'
            }
        }

        stage('Verify Deployment') {
            agent {
                label 'built-in'
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: "${SSH_CREDENTIALS}",
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=accept-new \
                        "$SSH_USER@$DEPLOY_SERVER_IP" \
                        "curl -f http://localhost:9090"
                    '''
                }

                echo 'Website verification completed'
            }
        }
    }

    post {
        success {
            echo 'CI/CD PIPELINE SUCCESSFUL!'
            echo 'Website deployed successfully.'
        }

        failure {
            echo 'CI/CD PIPELINE FAILED!'
            echo 'Check Jenkins Console Output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
