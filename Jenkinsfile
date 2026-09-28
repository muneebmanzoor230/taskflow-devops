pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t taskflow:ci .'
            }
        }
    }

    post {

        success {
            echo 'TaskFlow CI pipeline completed successfully!'
        }

        failure {
            echo 'TaskFlow CI pipeline failed!'
        }

    }
}
