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
                    # 1. Tự động gia hạn ECR Secret trên Kubernetes trước khi Deploy
                    kubectl create secret docker-registry ecr-secret \
                      --docker-server=${REGISTRY_URL} \
                      --docker-username=AWS \
                      --docker-password=\$(aws ecr get-login-password --region ${AWS_REGION}) \
                      --namespace=default \
                      --dry-run=client -o yaml | kubectl apply -f -

                    # 2. Thay đổi tag image trong deployment file
                    sed -i 's|${REGISTRY_URL}/${APP_NAME}:latest|${REGISTRY_URL}/${APP_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml

                    # 3. Apply manifests
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    # 4. Ép Kubernetes khởi động lại Pod với image/token mới
                    kubectl rollout restart deployment/nam-viettech-app
                """
            }
        }
    }
}
