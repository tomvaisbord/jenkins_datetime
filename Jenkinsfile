@Library('jenkins-shared-library@main') _  // Import the shared library

pipeline {
    agent { label 'agent_01' }

    environment {
        DOCKER_IMAGE = 'tomvais/datetime_app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                buildDocker("${DOCKER_IMAGE}:${BUILD_NUMBER}")
            }
        }
        
        stage('Test Docker Image') {
            steps {
                testDocker("${DOCKER_IMAGE}:${BUILD_NUMBER}")
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            when {
                expression { currentBuild.result == 'SUCCESS' }
            }
            steps {
                pushDocker("${DOCKER_IMAGE}:${BUILD_NUMBER}", 'docker-hub-api-token')
            }
        }
        
        stage('Deploy Docker Image') {
            when {
                expression { currentBuild.result == 'SUCCESS' }
            }
            steps {
                deployDocker("${DOCKER_IMAGE}:${BUILD_NUMBER}")
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Remove all containers except for the running `datetime-app`
            sh '''
                docker ps -aq --filter name!=datetime-app | xargs -r docker rm -f
                docker image prune -f
                docker volume prune -f
                docker network prune -f
            '''
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Cleaning up resources.'
        }
    }
}
