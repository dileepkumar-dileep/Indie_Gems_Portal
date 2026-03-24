pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "indie-gems-container"
        PORT = "9676"
        DOCKERHUB_USER = "diileepkumar"
        DOCKER_CREDS = "docker-cred"
        CONTAINER_PORT = "80"
        AWS_REGION = "eu-north-1"
        EKS_CLUSTER = "mycluster"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {
        stage('Checkout Code') {
            steps {
                dir("${WORK_DIR}") {
                    git branch: 'main', url: 'https://github.com/dileepkumar-dileep/Indie_Gems_Portal.git'
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
        }

        stage('Push Image to DockerHub') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Update K8s Image') {
            steps {
                sh '''
                sed -i "s|image:.*|image: $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yml
                '''
            }
        }

        stage('Configure EKS Access') {
            steps {
                sh '''
                export PATH=$PATH:/usr/local/bin
                aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                /usr/local/bin/kubectl config current-context
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yml
                kubectl apply -f k8s/service.yml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl rollout status deployment python-devops-app || true
                kubectl get pods -o wide
                kubectl get svc
                '''
            }
        }
    }
}
