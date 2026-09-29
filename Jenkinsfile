pipeline {
  agent any
  environment {
    IMAGE      = "roha_nnnn/webapp"
    KUBECONFIG = "/home/ec2-user/.kube/config"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Image') {
      steps {
        sh "sed -i 's/APP_VERSION/${BUILD_NUMBER}/' index.html"
        sh "docker build -t $IMAGE:${BUILD_NUMBER} -t $IMAGE:latest ."
      }
    }
    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'echo $P | docker login -u $U --password-stdin'
          sh "docker push $IMAGE:${BUILD_NUMBER}"
          sh "docker push $IMAGE:latest"
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh "sed -i 's#DOCKERUSER/webapp:latest#$IMAGE:${BUILD_NUMBER}#' k8s/deployment.yaml"
        sh "kubectl apply -f k8s/deployment.yaml"
        sh "kubectl rollout status deployment/webapp --timeout=120s"
      }
    }
  }
}
