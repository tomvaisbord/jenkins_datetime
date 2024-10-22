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
                testDocker('datetime-app')
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
}
