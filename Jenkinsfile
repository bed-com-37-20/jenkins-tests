pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = 'banda4hub/jenkins-hub'
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_HUB_CREDENTIALS_ID = 'docker-user'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    // Check if package.json exists
                    if (fileExists('package.json')) {
                        sh 'npm ci --only=production'
                    } else {
                        echo 'No package.json found, skipping npm install'
                    }
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Run tests if they exist
                    if (fileExists('package.json')) {
                        sh 'npm test || true'  // Continue even if tests fail
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG} ${DOCKER_HUB_REPO}:latest"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub using Jenkins credentials
                    withCredentials([usernamePassword(
                        credentialsId: DOCKER_HUB_CREDENTIALS_ID,
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                        sh "docker push ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB_REPO}:latest"
                        sh 'docker logout'
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo "Image pushed: ${DOCKER_HUB_REPO}:${DOCKER_IMAGE_TAG}"
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Clean up Docker images to save space
            sh '''
                docker images -f "dangling=true" -q | xargs --no-run-if-empty docker rmi || true
                docker system prune -f || true
            '''
        }
    }
}
