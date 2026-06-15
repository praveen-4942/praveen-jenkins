pipeline {
    agent {
        label 'devops-agent'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checkout Stage'
            }
        }

        stage('Validation') {
            steps {
                sh 'python --version || true'
            }
        }

        stage('Build') {
            steps {
                echo 'Build Success'
            }
        }
    }
}
