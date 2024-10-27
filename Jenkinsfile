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
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ./app'
                }
            }
        }
        
        stage('Remove Existing Test Container') {
            steps {
                sh 'docker rm -f datetime_app_test || true'
            }
        }

        stage('Run Docker Container for Testing') {
            steps {
                sh '''
                    docker run -d --name datetime_app_test -p 5000:5000 ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    sleep 5  # Wait for the app to start
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    try {
                        sh 'docker exec datetime_app_test curl -f http://localhost:5000'
                        currentBuild.result = 'SUCCESS'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        
        stage('Stop and Remove Test Container') {
            steps {
                sh 'docker stop datetime_app_test && docker rm datetime_app_test'
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            when {
                expression { currentBuild.result == 'SUCCESS' }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-api-token', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            docker login -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker logout
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up....'
            sh '''
                docker container prune -f
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
