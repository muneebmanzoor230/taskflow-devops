pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '484733237198'
        ECR_REPOSITORY = 'taskflow'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

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

        stage('ECR Login') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-taskflow']
                ]) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} |
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Docker Tag') {
            steps {
                sh '''
                    docker tag taskflow:ci \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Push') {
            steps {
                retry(3) {
                    sh '''
                        docker push \
                        ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                    '''
                }
            }
        }
    }

    post {

        success {
            echo 'TaskFlow CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'TaskFlow CI/CD pipeline failed!'
        }

        always {
            sh 'docker logout ${ECR_REGISTRY} || true'
        }
    }
}
