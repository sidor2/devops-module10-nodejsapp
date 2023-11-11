
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Webhook Received') {
            steps {
                echo 'Jenkins server received the webhook 10'
            }
        }
    }
}
