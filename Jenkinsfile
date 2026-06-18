def sendGchatNotification(message) {
    withCredentials([string(credentialsId: 'gchat-webhook-url', variable: 'WEBHOOK_URL')]) {
        sh """
        curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"text":"${message}"}' \
        "$WEBHOOK_URL"
        """
    }
}

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
                }
            }
        }

        stage('Build Started Notification') {
            steps {
                script {
                    sendGchatNotification("""
🚀 Build Started

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Branch: main
Environment: ${params.ENVIRONMENT}
Timestamp: ${new Date()}
""")
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
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl apply -f deployment-temp.yaml -n $NAMESPACE
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get deploy -n $NAMESPACE
                kubectl get pods -n $NAMESPACE
                '''
            }
        }

    }

    post {

        success {
            script {

                def commitId = sh(
                    script: 'git rev-parse --short HEAD',
                    returnStdout: true
                ).trim()

                sendGchatNotification("""
✅ Build Success

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Branch: main
Commit ID: ${commitId}
Environment: ${params.ENVIRONMENT}
Build URL: ${BUILD_URL}
Timestamp: ${new Date()}
""")
            }

            echo "Successfully deployed to ${params.ENVIRONMENT}"
        }

        failure {
            script {

                sendGchatNotification("""
❌ Build Failed

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
Branch: main
Environment: ${params.ENVIRONMENT}
Build URL: ${BUILD_URL}
Timestamp: ${new Date()}
""")
            }

            echo "Pipeline Failed"
        }
    }
}
