pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "gcr.io/YOUR_PROJECT_ID/my-app:${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/YOUR_USER/demo-k8s.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh 'docker build -t $DOCKER_IMAGE .'
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        script {
          sh 'docker push $DOCKER_IMAGE'
        }
      }
    }

    stage('Update Manifests') {
      steps {
        script {
          sh '''
            sed -i "s|image: .*|image: $DOCKER_IMAGE|" manifests/deployment.yaml
            git config user.email "ci@yourdomain.com"
            git config user.name "CI Bot"
            git commit -am "Update image to $DOCKER_IMAGE"
            git push origin main
          '''
        }
      }
    }
  }
}
