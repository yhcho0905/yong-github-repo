pipeline {
  agent any

  environment {
    AWS_REGION = "ap-northeast-2"
    ECR_REPO = "576714096343.dkr.ecr.ap-northeast-2.amazonaws.com/yong-my-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout Application') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Image') {
      steps {
        sh """
        docker build -t ${ECR_REPO}:${IMAGE_TAG} .
        docker push ${ECR_REPO}:${IMAGE_TAG}
        """
      }
    }

    stage('Update GitOps Repo') {
      steps {
        sh """
        git clone https://github.com/yhcho0905/yong-gitops-demo.git
        cd yong-gitops-demo/app

        sed -i 's|image:.*|image: ${ECR_REPO}:${IMAGE_TAG}|' deployment.yaml

        git config user.name "jenkins"
        git config user.email "jenkins@example.com"

        git add deployment.yaml
        git commit -m "update image ${IMAGE_TAG}"
        git push
        """
      }
    }

  }
}

