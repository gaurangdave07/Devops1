pipeline {
    agent any

    environment {
        DOCKER_REGISTRY_USER = 'your-dockerhub-username' // Change this
        IMAGE_NAME           = 'devops-node-app'
        IMAGE_TAG            = "${BUILD_NUMBER}"
    }

    stages {
        stage('Cloning Source') {
            steps {
                checkout scm
            }
        }

        stage('Lint & Test') {
            steps {
                echo 'Running tests...'
                // If you have npm tests, uncomment below:
                // sh 'npm install && npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker build -t ${DOCKER_REGISTRY_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh "docker push ${DOCKER_REGISTRY_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_REGISTRY_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                echo 'Deploying application container...'
                // If deploying to the same server/local Docker host:
                sh "docker stop ${IMAGE_NAME} || true"
                sh "docker rm ${IMAGE_NAME} || true"
                sh "docker run -d -p 3000:3000 --name ${IMAGE_NAME} ${DOCKER_REGISTRY_USER}/${IMAGE_NAME}:latest"
            }
        }
    }

    post {
        always {
            sh "docker logout"
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
