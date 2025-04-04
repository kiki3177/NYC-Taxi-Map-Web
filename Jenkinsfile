pipeline {
    agent any
    
    environment {
        AZURE_ACR_NAME = 'acrevewang1'
        IMAGE_NAME = 'capstone-repo-test-eve-3'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Implicit 'latest' tag
                    sh "docker build . -t ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                }
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'azure-acr-service-principal',  // Match the ID you set in Jenkins
                            usernameVariable: 'AZURE_CLIENT_ID',
                            passwordVariable: 'AZURE_CLIENT_SECRET'
                        )
                    ]) {
                        // Azure login
                        sh "az login --service-principal -u \$AZURE_CLIENT_ID -p \$AZURE_CLIENT_SECRET --tenant f6b6dd5b-f02f-441a-99a0-162ac5060bd2"
                        
                        // ACR login + Docker push
                        sh "az acr login --name ${AZURE_ACR_NAME}"
                        sh "docker push ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                    }
                }
            }
        }
                
        stage('Deploy') {
            steps {
                script {
                    sh "docker pull ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}"
                    sh "docker rm -f ${IMAGE_NAME} || true"
                    sh """
                        docker run -d \
                        -p 2003:80 \
                        --name ${IMAGE_NAME} \
                        ${AZURE_ACR_NAME}.azurecr.io/${IMAGE_NAME}
                    """
                }
            }
        }
    }
}