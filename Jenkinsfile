pipeline {
    agent {
        label 'agent'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_USER = "nafiya12"

        FRONTEND_IMAGE = "nafiya12/taskflow-frontend:latest"
        BACKEND_IMAGE  = "nafiya12/taskflow-backend:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Nafiya-26/nodejs-3tier.git'
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                set -eux

                git --version
                sudo docker --version
                sudo docker compose version
                sudo docker info
                '''
            }
        }
        stage('SonarQube Scan') {
    steps {
        script {
            def scannerHome = tool 'sonar-scanner'

            withSonarQubeEnv('sonarqube') {
                withCredentials([
                    string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')
                ]) {
                    sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }
    }
}

        stage('Build Docker Images') {
            steps {
                sh '''
                set -eux

                sudo docker build -t ${FRONTEND_IMAGE} ./frontend
                sudo docker build -t ${BACKEND_IMAGE} ./backend
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    string(credentialsId: 'dockerhub-username', variable: 'DOCKER_USERNAME'),
                    string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')
                ]) {
                    sh '''
                    echo "$DOCKER_TOKEN" | sudo docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                set -eux

                sudo docker push ${FRONTEND_IMAGE}
                sudo docker push ${BACKEND_IMAGE}
                '''
            }
        }
        stage('Trigger CD Pipeline') {

            steps {

                build job: '3tier-cd',
                    wait: true

            }

        }
    }

    post {

        success {
            echo "========================================"
            echo "CI Pipeline Executed Successfully"
            echo "========================================"
        }

        failure {
            echo "Pipeline Failed"
        }

        always {
            sh '''
            echo "===== Docker Images ====="
            sudo docker images
            '''
        }
    }
}