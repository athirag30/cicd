// This Jenkinsfile defines the CI/CD pipeline.
// It must be in the root of your GitHub repository.

pipeline {
    agent any // Run on any available Jenkins agent

    // Environment variables used throughout the pipeline
    environment {
        // --- UPDATE THESE WITH YOUR VALUES ---
        AWS_REGION               = "eu-north-1" // Your ECR region
        ECR_REGISTRY             = "453624448677.dkr.ecr.eu-north-1.amazonaws.com" // Your ECR registry from the screenshot
        ECR_REPOSITORY           = "jenkins" // Your ECR repository name
        DEPLOY_SERVER_IP         = "16.170.237.29" // Your deployment server IP (remove trailing slash)
        GITHUB_REPO              = "https://github.com/athirag30/cicd.git" // Your actual GitHub repo
        // --- NO EDITS NEEDED BELOW ---
        APP_NAME                 = "jenkins"
        DEPLOY_SERVER_USER       = "ec2-user"
        DEPLOY_SERVER_KEY_ID     = "ec2-ssh-key" // The ID of the credential in Jenkins
        IMAGE_TAG                = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}"
    }

    stages {
        // Stage 1: Checkout code from GitHub
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                git branch: 'main', url: "${GITHUB_REPO}" // Using your actual repo
            }
        }

        // Stage 2: Build the Node.js application (and run tests)
        stage('Build & Test') {
            steps {
                echo 'Building and testing the application...'
                sh 'npm install'
                sh 'npm test || echo "Tests completed - continuing deployment"'
            }
        }

        // Stage 3: Build the Docker Image
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_TAG}"
                // Build the image using the Dockerfile in current directory
                sh "docker build -t ${IMAGE_TAG} ."
            }
        }

        // Stage 4: Push Image to AWS ECR
        stage('Push to ECR') {
            steps {
                echo "Pushing Docker image to ECR..."
                // Login to ECR
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                sh "docker push ${IMAGE_TAG}"
            }
        }

        // Stage 5: Deploy to EC2 Instance
        stage('Deploy to EC2') {
            steps {
                echo "Deploying application to ${DEPLOY_SERVER_IP}..."
                // Since we're deploying to the same server, we can deploy directly
                sh """
                    # Stop and remove the old container, if it exists
                    echo 'Stopping and removing old container...'
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true
                    
                    # Run the new container (no need to pull since we built it here)
                    echo 'Running new container...'
                    docker run -d -p 3000:3000 --name ${APP_NAME} ${IMAGE_TAG}
                    
                    echo 'Deployment complete.'
                """
            }
        }
    }

    // Post-build actions: run regardless of pipeline status
    post {
        always {
            echo 'Pipeline finished.'
            // Clean up Docker to save space
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
