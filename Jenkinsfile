@Library('jenkins-shared-library@main') _  // Import the shared library

pipeline {
    agent { label 'agent_01' }

    environment {
        DOCKER_HUB_API_TOKEN = credentials('docker-hub-api-token')  // DockerHub credentials
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                buildDocker('datetime-app')
            }
        }
        stage('Test Docker Image') {
            steps {
                echo "Testing Docker Image datetime-app..."
                script {
                    // Stop and remove any existing test container
                    sh '''
                    if [ $(docker ps -aq -f name=test-container) ]; then
                        docker stop test-container
                        docker rm test-container
                    fi
                    '''

                    // Run and test the new container
                    sh '''
                    docker run --name test-container -d datetime-app
                    docker exec test-container curl http://localhost:5000
                    docker stop test-container
                    docker rm test-container
                    '''
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                pushDocker('datetime-app', 'docker-hub-api-token')
            }
        }
        stage('Deploy Docker Image') {
            steps {
                deployDocker('datetime-app')
            }
        }
    }

    post {
        always {
            echo 'Cleaning up....'
            // Remove any stopped containers
            sh 'docker container prune -f'
            // Remove dangling images
            sh 'docker image prune -f'
            // Optionally remove specific images if needed
            sh 'docker rmi -f datetime-app || true'
            // Remove unused volumes (if applicable)
            sh 'docker volume prune -f'
            // Remove unused networks (if applicable)
            sh 'docker network prune -f'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Cleaning up resources.'
        }
    }
}
