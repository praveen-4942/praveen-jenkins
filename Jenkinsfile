pipeline {
agent {
label 'kaniko'
}

```
environment {
    AWS_ACCOUNT_ID = '130759691668'
    AWS_REGION = 'us-east-1'
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
            sh 'python3 -m py_compile app/app.py'
        }
    }

    stage('Build and Push Image') {
        steps {
            container('kaniko') {
                sh '''
                /kaniko/executor \
                --context=$WORKSPACE \
                --dockerfile=$WORKSPACE/Dockerfile \
                --destination=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:$IMAGE_TAG \
                --destination=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO:latest
                '''
            }
        }
    }
}

post {
    success {
        echo 'Image Built and Pushed Successfully'
    }

    failure {
        echo 'Pipeline Failed'
    }
}
```

}
