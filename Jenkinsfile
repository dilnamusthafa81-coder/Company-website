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
}

post {
    success {
        echo 'Docker image successfully built and pushed to Docker Hub!'
    }

    failure {
        echo 'Pipeline failed. Check the console output.'
    }
}

}
