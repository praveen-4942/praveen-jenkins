
pipeline {

    agent {
        label 'jenkins-jenkins-agent'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'qa', 'uat'],
            description: 'Select deployment environment'
        )
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '130759691668'
        IMAGE_REPO = 'praveenkumarg/app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Set Environment') {
            steps {
                script {
                    if (params.ENVIRONMENT == 'dev') {
                        env.REPLICAS = "1"
                    } else if (params.ENVIRONMENT == 'qa') {
                        env.REPLICAS = "2"
                    } else {
                        env.REPLICAS = "3"
                    }

                    env.NAMESPACE = params.ENVIRONMENT

                    echo "Environment : ${env.NAMESPACE}"
                    echo "Replicas    : ${env.REPLICAS}"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Code Validation') {
            steps {
                sh 'python3 -m py_compile app/app.py'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build \
                -t $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG .

                docker tag \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:latest
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

                docker push \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:latest
                '''
            }
        }

        stage('Prepare Deployment') {
            steps {
                sh '''
                sed "s|IMAGE_NAME|$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG|g" deployment.yaml \
                | sed "s|REPLICA_COUNT|$REPLICAS|g" \
                > deployment-temp.yaml

                echo "Generated Deployment File:"
                cat deployment-temp.yaml
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl apply -f deployment-temp.yaml -n $NAMESPACE

                kubectl delete svc demo-app-service \
                -n $NAMESPACE \
                --ignore-not-found=true

                kubectl apply -f service.yaml -n $NAMESPACE
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get deploy -n $NAMESPACE
                kubectl get svc -n $NAMESPACE
                kubectl get pods -n $NAMESPACE
                '''
            }
        }

    }

    post {

        success {
            echo "Successfully deployed to ${params.ENVIRONMENT}"
        }

        failure {
            echo "Pipeline Failed"
        }

    }
}
