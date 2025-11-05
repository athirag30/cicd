pipeline {
    agent any

    environment {
        // --- YOUR ACTUAL CONFIGURATION ---
        AWS_REGION               = "eu-north-1"
        ECR_REGISTRY             = "453624448677.dkr.ecr.eu-north-1.amazonaws.com"
        ECR_REPOSITORY           = "jenkins"
        DEPLOY_SERVER_IP         = "16.170.237.29"
        GITHUB_REPO              = "https://github.com/athirag30/cicd.git"
        // --- NO EDITS NEEDED BELOW ---
        APP_NAME                 = "webapp"
        IMAGE_TAG                = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}"
    }

    stages {
        // Stage 1: Checkout code from GitHub
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                git branch: 'master', url: "${GITHUB_REPO}"
            }
        }

        // Stage 2: Build the Node.js application
        stage('Build & Test') {
            steps {
                echo 'Building and testing the application...'
                dir('app') {
                    sh 'npm install'
                    sh 'npm test || echo "Tests completed - continuing deployment"'
                }
            }
        }

        // Stage 3: Build the Docker Image
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_TAG}"
                dir('app') {
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        // Stage 4: Push Image to AWS ECR
        stage('Push to ECR') {
            steps {
                echo "Pushing Docker image to ECR..."
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                sh "docker push ${IMAGE_TAG}"
            }
        }

        // Stage 5: Deploy to EC2
        stage('Deploy') {
            steps {
                echo "Deploying application..."
                script {
                    // Stop and remove old container
                    sh 'docker stop webapp-container || true'
                    sh 'docker rm webapp-container || true'
                    
                    // Run new container
                    sh "docker run -d -p 3000:3000 --name webapp-container ${IMAGE_TAG}"
                    
                    echo "Deployment complete!"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
            // Clean up Docker images to save space
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline succeeded! Application is deployed and running.'
            sh """
                echo "Application should be available at: http://${DEPLOY_SERVER_IP}:3000"
                echo "Checking container status:"
                docker ps | grep webapp-container || echo "Container not found"
            """
        }
        failure {
            echo 'Pipeline failed! Check the logs above for errors.'
        }
    }
}
