pipeline {
    agent {
        kubernetes {
            label 'docker-agent'
            defaultContainer 'docker'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24.0.6-dind
    command:
    - cat
    tty: true
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-graph-storage
      mountPath: /var/lib/docker
  - name: jnlp
    image: jenkins/inbound-agent:3301.v4363ddcca_4e7-3
  volumes:
  - name: docker-graph-storage
    emptyDir: {}
"""
        }
    }

    environment {
        IMAGE = "maheshvenkata133/telugu-evolution:${BUILD_NUMBER}"
        GIT_CREDENTIALS_ID = 'github-token'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/Maheshvenkata133/demo-k8s.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'dockerd-entrypoint.sh & sleep 20'
                sh 'docker version'
                sh 'docker build -t $IMAGE .'
            }
        }

        stage('Push to Container Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo $PASSWORD | docker login -u $USERNAME --password-stdin
                        docker push $IMAGE
                    '''
                }
            }
        }

        stage('Update Deployment') {
            steps {
                sh 'sed -i "s|image: .*|image: $IMAGE|" manifests/deployment.yaml'
                sh 'git config user.email "jenkins@example.com"'
                sh 'git config user.name "Jenkins CI"'
                sh 'git commit -am "Updated image to $IMAGE" || echo "No changes to commit"'
                sh 'git push origin main'
            }
        }
    }
}
