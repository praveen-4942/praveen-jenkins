pipeline {
    agent {
        label 'devops-agent'
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '130759691668'
        IMAGE_REPO = 'praveenkumarg/app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Code Validation') {
            steps {
                sh '''
                python -m py_compile app/app.py
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy image --exit-code 1 --severity CRITICAL \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Push To ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo 'Image pushed successfully to ECR'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
