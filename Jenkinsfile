pipeline {
    agent any

    environment {
        IMAGE_NAME = "nodejs-docker-app"
        CONTAINER_NAME = "nodejs-container"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning source code from GitHub..."
                git branch: 'master', url: 'https://github.com/Mohamed-Elsayed-Hussein/Devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('Docker/Session 3 Dockerize a Node.js Application') {
                    echo "Building Docker image from Dockerfile..."
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Run Container') {
            steps {
                echo 'Running Docker container...'
                script {
                    // Stop old container if it exists
                    sh "docker rm -f ${IMAGE_NAME} || true"

                    // Run the container
                    sh """
                        docker run -d -p 3000:3000 --name ${IMAGE_NAME} ${IMAGE_NAME}:${BUILD_NUMBER}}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ App deployed successfully on port 3000."
        }
        failure {
            echo "❌ Pipeline failed! Check logs for details."
        }
    }
}

