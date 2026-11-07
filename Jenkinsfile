pipeline {
    agent any

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

    }
}