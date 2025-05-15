pipeline {
    agent any

    environment {
        AZURE_ACR_NAME = 'labacrdevops3'
        IMAGE_NAME_FRONTEND = 'opendevops-nyctaxiweb-frontend'
        IMAGE_NAME_BACKEND = 'opendevops-nyctaxiweb-backend'
        ACR_LOGIN_SERVER = "${AZURE_ACR_NAME}.azurecr.io"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Frontend & Backend') {
            steps {
                dir('front_end') {
                    sh 'docker build -t ${IMAGE_NAME_FRONTEND} .'
                    sh 'docker tag ${IMAGE_NAME_FRONTEND} ${ACR_LOGIN_SERVER}/${IMAGE_NAME_FRONTEND}'
                }
                dir('back_end') {
                    sh 'docker build -t ${IMAGE_NAME_BACKEND} .'
                    sh 'docker tag ${IMAGE_NAME_BACKEND} ${ACR_LOGIN_SERVER}/${IMAGE_NAME_BACKEND}'
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'jenkins-acr-scope-map-cred',  // Match the ID you set in Jenkins
                            usernameVariable: 'ACR_USER',
                            passwordVariable: 'ACR_PASS'
                        )
                    ]) {
                        sh """
                            docker login ${ACR_LOGIN_SERVER} -u \$ACR_USER -p \$ACR_PASS
                            docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME_FRONTEND}
                            docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME_BACKEND}
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker pull ${ACR_LOGIN_SERVER}/${IMAGE_NAME_FRONTEND}'
                sh 'docker pull ${ACR_LOGIN_SERVER}/${IMAGE_NAME_BACKEND}'
                sh 'docker rm -f frontend || true'
                sh 'docker rm -f backend || true'
                sh 'docker run -d --name frontend -p 8080:80 ${ACR_LOGIN_SERVER}/${IMAGE_NAME_FRONTEND}'
                sh 'docker run -d --name backend -p 3000:3000 ${ACR_LOGIN_SERVER}/${IMAGE_NAME_BACKEND}'
            }
        }
    }
}