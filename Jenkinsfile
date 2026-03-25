pipeline {
    agent any

    environment {
        WORK_DIR = "/var/lib/jenkins/workspace/Game"
        IMAGE_NAME = "indie-gems"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USER = "diileepkumar"
        DOCKER_CREDS = "docker-cred"
        KUBECONFIG_CREDENTIAL_ID = "kubeconfig"
        EMAIL_ID = "yourmail@gmail.com"
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
                        docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Push Image') {
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
        }

        stage('Deploy') {
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
        }
    }
}
