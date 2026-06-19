@Library('shared-lib') _

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

            if (env.BRANCH_NAME == "dev") {
                env.NAMESPACE = "dev"
                env.REPLICAS = "1"
            }

            else if (env.BRANCH_NAME == "release") {
                env.NAMESPACE = "qa"
                env.REPLICAS = "2"
            }

            else {
                env.NAMESPACE = "uat"
                env.REPLICAS = "3"
            }

            echo "Branch : ${env.BRANCH_NAME}"
            echo "Namespace : ${env.NAMESPACE}"
            echo "Replicas : ${env.REPLICAS}"
        }
    }
}
            steps {
                script {

                    if (env.BRANCH_NAME == "dev") {
                        env.NAMESPACE = "dev"
                        env.REPLICAS = "1"
                    }
                    else if (env.BRANCH_NAME == "release") {
                        env.NAMESPACE = "qa"
                        env.REPLICAS = "2"
                    }
                    else {
                        env.NAMESPACE = "uat"
                        env.REPLICAS = "3"
                    }

                    echo "Branch : ${env.BRANCH_NAME}"
                    echo "Namespace : ${env.NAMESPACE}"
                    echo "Replicas : ${env.REPLICAS}"
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
<<<<<<< HEAD
Branch: main
Environment: ${params.ENVIRONMENT}
=======
Branch: ${env.BRANCH_NAME}
Environment: ${env.NAMESPACE}
>>>>>>> c210a4d (Bonus1 multibranch support)
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
                dockerBuild()
            }
        }

        stage('Push To ECR') {
            steps {
                ecrPush()
            }
        }

        stage('Deploy to EKS') {
            steps {
                k8sDeploy()
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
<<<<<<< HEAD
Branch: main
Commit ID: ${commitId}
Environment: ${params.ENVIRONMENT}
=======
Branch: ${env.BRANCH_NAME}
Commit ID: ${commitId}
Environment: ${env.NAMESPACE}
>>>>>>> c210a4d (Bonus1 multibranch support)
Build URL: ${BUILD_URL}
Timestamp: ${new Date()}
""")
            }

            echo "Successfully deployed to ${params.ENVIRONMENT}"
            echo "Successfully deployed to ${env.NAMESPACE}"
        }

        failure {
            script {

                sendGchatNotification("""
❌ Build Failed

Job: ${JOB_NAME}
Build: #${BUILD_NUMBER}
<<<<<<< HEAD
Branch: main
Environment: ${params.ENVIRONMENT}
=======
Branch: ${env.BRANCH_NAME}
Environment: ${env.NAMESPACE}
>>>>>>> c210a4d (Bonus1 multibranch support)
Build URL: ${BUILD_URL}
Timestamp: ${new Date()}
""")
            }

            echo "Pipeline Failed"
            echo "Pipeline Failed for ${env.NAMESPACE}"
        }
    }
}
