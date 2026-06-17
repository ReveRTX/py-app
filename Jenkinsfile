pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'flask-calculator:latest'
        DOCKERHUB_USERNAME = credentials('docker-username')
        DOCKERHUB_ACCESS_TOKEN = credentials('docker-access-token')
    }

    stages {
        stage('Checkout/source') {
            steps {
                // Clone the repository containing your Flask calculator application
                git 'https://github.com/ReveRTX/py-app'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image for the Flask application
                    sh "sudo docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run unit tests inside the Docker container
                    sh """
                    sudo docker run --name test-container ${DOCKER_IMAGE} /bin/sh -c 'python -m unittest discover -s /app/tests'
                    sudo docker rm test-container
                    """
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    // Login to Docker Hub
                    sh """
                    echo ${DOCKERHUB_ACCESS_TOKEN} | sudo docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                    # Tag and push the Docker image
                    sudo docker tag ${DOCKER_IMAGE} ${DOCKERHUB_USERNAME}/${DOCKER_IMAGE}
                    sudo docker push ${DOCKERHUB_USERNAME}/${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the application by running the Docker container
                    sh """
                    sudo docker stop py-app && sudo docker remove py-app || true
                    sudo docker run -d -p 5000:80 --name py-app ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
