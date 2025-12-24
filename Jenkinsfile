pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-demo"
        DOCKER_REPO = "houssamechchari/devops-demo"
        GITEA_URL = "http://20.199.168.227:3000/azureuser/devops-demo.git"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker tag $IMAGE_NAME $DOCKER_USER/$IMAGE_NAME:latest
                    docker push $DOCKER_USER/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Push Code to Gitea') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'gitea-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh '''
                    git remote remove gitea || true
                    git remote add gitea http://$GIT_USER:$GIT_PASS@20.199.168.227:3000/azureuser/devops-demo.git
                    git fetch origin
                    git checkout -B main origin/main
                    git push gitea main --force
                    '''
                }
            }
        }
    }
}
