pipeline {
agent any

```
environment {
    WORK_DIR = "/var/lib/jenkins/workspace/Game"
    IMAGE_NAME = "indie-gems"
    IMAGE_TAG = "${BUILD_NUMBER}"
    DOCKERHUB_USER = "diileepkumar"
    DOCKER_CREDS = "docker-cred"
    KUBECONFIG_CREDENTIAL_ID = "kubeconfig"
    EMAIL_ID = "yourmail@gmail.com"   // <-- change this
}

stages {

    stage('Checkout Code') {
        steps {
            dir("${WORK_DIR}") {
                git branch: 'main', url: 'https://github.com/dileepkumar-dileep/Indie_Gems_Portal.git'
            }
        }
        post {
            success {
                mail to: "${EMAIL_ID}",
                     subject: "SUCCESS: Checkout Stage - Build #${BUILD_NUMBER}",
                     body: "Checkout completed successfully."
            }
            failure {
                mail to: "${EMAIL_ID}",
                     subject: "FAILED: Checkout Stage - Build #${BUILD_NUMBER}",
                     body: "Checkout failed. Please check Jenkins."
            }
        }
    }

    stage('Build Docker Image') {
        steps {
            dir("${WORK_DIR}") {
                sh '''
                    docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }
        post {
            success {
                mail to: "${EMAIL_ID}",
                     subject: "SUCCESS: Docker Build - Build #${BUILD_NUMBER}",
                     body: "Docker image built successfully."
            }
            failure {
                mail to: "${EMAIL_ID}",
                     subject: "FAILED: Docker Build - Build #${BUILD_NUMBER}",
                     body: "Docker build failed."
            }
        }
    }

    stage('DockerHub Login & Push') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: "${DOCKER_CREDS}",
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
        post {
            success {
                mail to: "${EMAIL_ID}",
                     subject: "SUCCESS: Docker Push - Build #${BUILD_NUMBER}",
                     body: "Image pushed to DockerHub."
            }
            failure {
                mail to: "${EMAIL_ID}",
                     subject: "FAILED: Docker Push - Build #${BUILD_NUMBER}",
                     body: "Docker push failed."
            }
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            dir("${WORK_DIR}") {
                withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIAL_ID}"]) {
                    sh '''
                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml
                    '''
                }
            }
        }
        post {
            success {
                mail to: "${EMAIL_ID}",
                     subject: "SUCCESS: Deployment - Build #${BUILD_NUMBER}",
                     body: "Application deployed successfully."
            }
            failure {
                mail to: "${EMAIL_ID}",
                     subject: "FAILED: Deployment - Build #${BUILD_NUMBER}",
                     body: "Deployment failed."
            }
        }
    }

    stage('Verify Deployment') {
        steps {
            withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIAL_ID}"]) {
                sh '''
                kubectl rollout status deployment python-devops-app || true
                kubectl get pods -o wide
                kubectl get svc
                '''
            }
        }
        post {
            success {
                mail to: "${EMAIL_ID}",
                     subject: "SUCCESS: Verification - Build #${BUILD_NUMBER}",
                     body: "Deployment verified successfully."
            }
            failure {
                mail to: "${EMAIL_ID}",
                     subject: "FAILED: Verification - Build #${BUILD_NUMBER}",
                     body: "Verification failed."
            }
        }
    }
}

post {
    always {
        mail to: "${EMAIL_ID}",
             subject: "Build Completed: #${BUILD_NUMBER}",
             body: "Check full pipeline result in Jenkins."
    }
}
```

}
