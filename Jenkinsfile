pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-1'
        ECR_REGISTRY = '640440227805.dkr.ecr.us-west-1.amazonaws.com'
        ECR_REPOSITORY = 'devsecops-ecommerce'
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('Docker Build') {
            steps {
                sh 'docker build -t devsecops-ecommerce:latest .'
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --ignore-unfixed \
                    --exit-code 1 \
                    devsecops-ecommerce:latest
                '''
            }
        }

        stage('ECR Push') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_REGISTRY}

                    docker tag devsecops-ecommerce:latest \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    docker push \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

    }
}