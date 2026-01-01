pipeline {
    agent any

    environment {
        GITEA_REGISTRY = "20.199.168.227:3000"
        GITEA_NAMESPACE = "azureuser"
        IMAGE_NAME = "devops-demo"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout source code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Tag image for Gitea Registry') {
            steps {
                sh '''
                  docker tag $IMAGE_NAME:$IMAGE_TAG \
                  $GITEA_REGISTRY/$GITEA_NAMESPACE/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Push image to Gitea Registry') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'gitea-registry-creds',
                    usernameVariable: 'GITEA_USER',
                    passwordVariable: 'GITEA_PASS'
                )]) {
                    sh '''
                      echo "$GITEA_PASS" | docker login $GITEA_REGISTRY \
                      -u "$GITEA_USER" --password-stdin

                      docker push \
                      $GITEA_REGISTRY/$GITEA_NAMESPACE/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Image Docker buildée et pushée vers Gitea avec succès"
        }
        failure {
            echo "❌ Erreur dans le pipeline"
        }
    }
}
