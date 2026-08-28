pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv jenkins-venv
                    . jenkins-venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . jenkins-venv/bin/activate
                    pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t jenkins-ci-cd-app:${BUILD_NUMBER} .
                    
                    docker tag \
                    jenkins-ci-cd-app:${BUILD_NUMBER} \
                    jenkins-ci-cd-app:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD pipeline failed!'
        }
    }
}
