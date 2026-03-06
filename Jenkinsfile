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
    volumeMounts:
    - name: workspace-volume
      mountPath: /home/jenkins/agent
  volumes:
  - name: workspace-volume
    emptyDir: {}
"""
    }
  }

  environment {
    AWS_REGION = "ap-northeast-2"
    ECR_REPO   = "576714096343.dkr.ecr.ap-northeast-2.amazonaws.com/yong-my-app"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout Source') {
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
        withCredentials([usernamePassword(
          credentialsId: 'yong-git-token-for-jenkins',
          usernameVariable: 'GIT_USERNAME',
          passwordVariable: 'GIT_PASSWORD'
        )]) {

          sh '''
          git config --global credential.helper store
          echo "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com" > ~/.git-credentials

          git clone https://github.com/yhcho0905/yong-gitops-demo.git
          cd yong-gitops-demo/app

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
}
