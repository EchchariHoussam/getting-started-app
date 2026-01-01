pipeline {
    agent any
//s
    environment {
        GITEA_REGISTRY = "20.199.168.227:3000"
        GITEA_NAMESPACE = "azureuser"
        IMAGE_NAME = "devops-demo"
        IMAGE_TAG = "latest"
        GITEA_REPO = "devops-demo"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Tag & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'gitea-registry-creds',
                    usernameVariable: 'GITEA_USER',
                    passwordVariable: 'GITEA_PASS'
                )]) {
                    sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${GITEA_REGISTRY}/${GITEA_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}
                    echo "${GITEA_PASS}" | docker login ${GITEA_REGISTRY} -u "${GITEA_USER}" --password-stdin
                    docker push ${GITEA_REGISTRY}/${GITEA_NAMESPACE}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Link Docker Package to Repo') {
            steps {
                withCredentials([string(credentialsId: 'gitea-token', variable: 'GITEA_TOKEN')]) {
                    sh """
                    curl -s -X POST -H "Authorization: token ${GITEA_TOKEN}" \
                         -H "Content-Type: application/json" \
                         -d '{ "name": "${IMAGE_NAME}", "package_type": "container", "link": { "repository": "${GITEA_NAMESPACE}/${GITEA_REPO}" } }' \
                         http://${GITEA_REGISTRY}/api/v1/packages/${GITEA_NAMESPACE}/container
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Docker image pushed and linked to Gitea repo successfully!"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
//ssss
//hhhh