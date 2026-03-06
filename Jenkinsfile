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

    stage('Build & Push Image') {
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

    stage('Update GitOps Repo') {
      steps {
        sh '''
        git clone https://github.com/yhcho0905/yong-gitops-demo.git
        cd yong-gitops-demo/app
        echo "1"

        sed -i "s|image:.*|image: ${ECR_REPO}:${IMAGE_TAG}|" deployment.yaml

        git config user.name "jenkins"
        git config user.email "jenkins@example.com"

        git add deployment.yaml
        git commit -m "update image ${IMAGE_TAG}"
        git push
        '''
      }
    }

  }
}
