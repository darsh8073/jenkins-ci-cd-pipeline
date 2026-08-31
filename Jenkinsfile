pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'darshan212/jenkins-ci-cd-app'
        DEPLOYMENT_ATTEMPTED = 'false'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'

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
                echo 'Running automated tests...'

                sh '''
                    . jenkins-venv/bin/activate
                    pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"

                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -t ${DOCKER_IMAGE}:latest \
                        .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker images to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    env.DEPLOYMENT_ATTEMPTED = 'true'
                }

                echo "Deploying version ${BUILD_NUMBER}..."

                sh '''
                    echo "Pulling image from Docker Hub..."

                    docker pull ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    echo "Stopping old container..."

                    docker stop jenkins-ci-cd-app || true
                    docker rm jenkins-ci-cd-app || true

                    echo "Starting new container..."

                    docker run -d \
                        --name jenkins-ci-cd-app \
                        -p 5000:5000 \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    echo "Deployment completed."
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application health...'

                sh '''
                    sleep 5

                    curl --fail --silent \
                        http://localhost:5000/health

                    echo ""
                    echo "Application health check PASSED."
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo ' CI/CD PIPELINE SUCCESSFUL'
            echo '======================================'
            echo "Deployed version: ${BUILD_NUMBER}"
            echo "Docker image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
        }

        failure {
            echo '======================================'
            echo ' CI/CD PIPELINE FAILED'
            echo '======================================'

            script {

                if (env.DEPLOYMENT_ATTEMPTED == 'true') {

                    if (env.BUILD_NUMBER.toInteger() > 1) {

                        def previousBuild = env.BUILD_NUMBER.toInteger() - 1

                        echo "Attempting rollback to version ${previousBuild}..."

                        sh """
                            echo "Stopping failed deployment..."

                            docker stop jenkins-ci-cd-app || true
                            docker rm jenkins-ci-cd-app || true

                            echo "Pulling previous image..."

                            docker pull ${DOCKER_IMAGE}:${previousBuild}

                            echo "Starting previous version..."

                            docker run -d \
                                --name jenkins-ci-cd-app \
                                -p 5000:5000 \
                                ${DOCKER_IMAGE}:${previousBuild}

                            echo "Rollback completed."
                        """

                    } else {

                        echo 'No previous build available for rollback.'

                    }

                } else {

                    echo 'Deployment was not attempted. Rollback not required.'

                }
            }
        }

        always {
            echo "Jenkins build number: ${BUILD_NUMBER}"
            echo "Pipeline execution completed."
        }
    }
}
