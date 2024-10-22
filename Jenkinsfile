pipeline {
    agent any
    
    triggers {
        githubPush() // Trigger the pipeline on a GitHub push event
    }
    
    stages {
        stage('Test Webhook') {
            steps {
                echo 'Hello, Jenkins! The webhook is working.'
            }
        }
    }
}
