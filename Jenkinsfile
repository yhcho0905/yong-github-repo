pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - cat
    tty: true
"""
    }
  }

  environment {
    AWS_REGION = "ap-northeast-2"
    ECR_REPO   = "576714096343.dkr.ecr.ap-northeast-2.amazonaws.com/yong-my-app"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context=. \
              --dockerfile=Dockerfile \
              --destination=${ECR_REPO}:${IMAGE_TAG} \
              --destination=${ECR_REPO}:latest \
              --verbosity=info
          '''
        }
      }
    }
  }
}
