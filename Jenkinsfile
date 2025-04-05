pipeline {
  agent any

  environment {
    IMAGE = "gcr.io/psec-dev/telugu-evolution:${BUILD_NUMBER}"   // change to your image
    GIT_CREDENTIALS_ID = 'github-token'   // set this in Jenkins
    DOCKER_CREDENTIALS_ID = 'gcr-json'    // set this in Jenkins
  }

  stages {
    stage('Clone Repo') {
      steps {
        git branch: 'main', url: 'https://github.com/Maheshvenkata133/demo-k8s.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Push to Container Registry') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh """
            echo $PASSWORD | docker login -u $USERNAME --password-stdin https://gcr.io
            docker push $IMAGE
          """
        }
      }
    }

    stage('Update deployment.yaml') {
      steps {
        script {
          def file = 'manifests/deployment.yaml'
          sh "sed -i 's|image: .*|image: ${IMAGE}|' ${file}"
        }
      }
    }

    stage('Push to GitHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
          sh """
            git config user.name "ci-bot"
            git config user.email "ci@example.com"
            git add manifests/deployment.yaml
            git commit -m "CI: update image to $IMAGE"
            git push https://${GIT_USER}:${GIT_PASS}@github.com/Maheshvenkata133/demo-k8s.git
          """
        }
      }
    }
  }
}
