pipeline {
  agent any
  environment {
    IMAGE = "dp7049/cicd-app"
    TAG   = "${BUILD_NUMBER}"
    PATH  = "/usr/local/bin:/opt/homebrew/bin:/Applications/Docker.app/Contents/Resources/bin:${PATH}"
    DOCKER_CONFIG = "${WORKSPACE}/.docker"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Image') {
      steps { sh 'docker build -t $IMAGE:$TAG .' }
    }
    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'mkdir -p $DOCKER_CONFIG'
          sh 'echo $P | docker login -u $U --password-stdin'
          sh 'docker push $IMAGE:$TAG'
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        sh '''
          kubectl config use-context docker-desktop
          sed "s|__IMAGE__|$IMAGE:$TAG|" k8s/deployment.yaml | kubectl apply -f -
          kubectl apply -f k8s/service.yaml
          kubectl rollout status deployment/cicd-app
        '''
      }
    }
  }
}
