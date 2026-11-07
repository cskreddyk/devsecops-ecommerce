pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-1'
        ECR_REGISTRY = '640440227805.dkr.ecr.us-west-1.amazonaws.com'
        ECR_REPOSITORY = 'devsecops-ecommerce'
    }

    stages {

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=devsecops-ecommerce \
                        -Dsonar.projectName=devsecops-ecommerce \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions='**/__pycache__/**,**/.venv/**,**/venv/**'
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                    -t ${ECR_REPOSITORY}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --ignore-unfixed \
                    --exit-code 1 \
                    ${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('ECR Push') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}

                    docker tag \
                    ${ECR_REPOSITORY}:${BUILD_NUMBER} \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker push \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name desecops-eks

                    kubectl set image deployment/devsecops-ecommerce \
                    devsecops-ecommerce=${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    kubectl rollout status deployment/devsecops-ecommerce \
                    --timeout=180s
                '''
            }
        }
    }
}