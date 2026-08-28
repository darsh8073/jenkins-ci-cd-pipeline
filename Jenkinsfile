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

        stage('Deploy') {
            steps {
                sh '''
                    docker stop jenkins-ci-cd-app || true
                    docker rm jenkins-ci-cd-app || true

                    docker run -d \
                    --name jenkins-ci-cd-app \
                    -p 5000:5000 \
                    jenkins-ci-cd-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 5

                    curl --fail http://localhost:5000

                    echo ""
                    echo "Application is healthy!"
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
