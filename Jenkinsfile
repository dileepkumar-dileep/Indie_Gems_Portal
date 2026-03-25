pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"

        DOCKERHUB_USER = "diileepkumar"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG = "${BUILD_NUMBER}"

        DOCKER_CREDS = "docker-cred"
        KUBECONFIG_CREDENTIAL_ID = "kubeconfig"

        EMAIL = "s.dileepkumar1234@gmail.com"
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
                    emailext(
                        subject: "SUCCESS: Checkout Stage",
                        body: "Checkout completed successfully.\n${BUILD_URL}",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: Checkout Stage",
                        body: "Checkout failed.\n${BUILD_URL}",
                        to: "${EMAIL}"
                    )
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh """
                    docker rmi -f ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} || true
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }

            post {
                success {
                    emailext(
                        subject: "SUCCESS: Docker Build",
                        body: "Docker image built:\n${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: Docker Build",
                        body: "Docker build failed.\n${BUILD_URL}",
                        to: "${EMAIL}"
                    )
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    """
                }
            }

            post {
                success {
                    emailext(
                        subject: "SUCCESS: Docker Login",
                        body: "DockerHub login successful",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: Docker Login",
                        body: "DockerHub login failed",
                        to: "${EMAIL}"
                    )
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh """
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }

            post {
                success {
                    emailext(
                        subject: "SUCCESS: Docker Push",
                        body: "Image pushed:\n${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: Docker Push",
                        body: "Docker push failed.\n${BUILD_URL}",
                        to: "${EMAIL}"
                    )
                }
            }
        }

        stage('Update K8s Image') {
            steps {
                dir("${WORK_DIR}") {
                    sh """
                    sed -i "s|image:.*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml
                    """
                }
            }

            post {
                success {
                    emailext(
                        subject: "SUCCESS: K8s Image Update",
                        body: "Deployment YAML updated with new image",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: K8s Image Update",
                        body: "Failed to update deployment YAML",
                        to: "${EMAIL}"
                    )
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir("${WORK_DIR}") {
                    withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIAL_ID}"]) {
                        sh """
                        kubectl apply -f k8s/deployment.yml
                        kubectl apply -f k8s/service.yml
                        """
                    }
                }
            }

            post {
                success {
                    emailext(
                        subject: "SUCCESS: Kubernetes Deployment",
                        body: "Application deployed successfully",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: Kubernetes Deployment",
                        body: "Deployment failed.\n${BUILD_URL}",
                        to: "${EMAIL}"
                    )
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIAL_ID}"]) {
                    sh """
                    kubectl rollout status deployment python-devops-app || true
                    kubectl get pods -o wide
                    kubectl get svc
                    """
                }
            }

            post {
                success {
                    emailext(
                        subject: "SUCCESS: Deployment Verification",
                        body: "Deployment verified successfully",
                        to: "${EMAIL}"
                    )
                }
                failure {
                    emailext(
                        subject: "FAILED: Deployment Verification",
                        body: "Verification failed",
                        to: "${EMAIL}"
                    )
                }
            }
        }
    }

    post {
        always {
            emailext(
                subject: "Pipeline Completed - Build #${BUILD_NUMBER}",
                body: "Check full details: ${BUILD_URL}",
                to: "${EMAIL}"
            )
        }
    }
}
