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