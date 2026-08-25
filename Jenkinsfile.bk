pipeline {
    agent any

    environment {
        AWS_REGION     = 'ap-southeast-2'
        ECR_ACCOUNT_ID = '475309741409'
        APP_NAME       = 'nam-web-app'
        REGISTRY_URL   = "${ECR_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${APP_NAME}:${IMAGE_TAG} ${REGISTRY_URL}/${APP_NAME}:${IMAGE_TAG}"
                sh "docker tag ${APP_NAME}:${IMAGE_TAG} ${REGISTRY_URL}/${APP_NAME}:latest"
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${REGISTRY_URL}"
                sh "docker push ${REGISTRY_URL}/${APP_NAME}:${IMAGE_TAG}"
                sh "docker push ${REGISTRY_URL}/${APP_NAME}:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    sed -i 's|${REGISTRY_URL}/${APP_NAME}:latest|${REGISTRY_URL}/${APP_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
    
    post {
        always {
            sh "docker logout ${REGISTRY_URL} || true"
        }
    }
}
