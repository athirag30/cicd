pipeline {
    agent any

    environment {
        AWS_REGION               = "eu-north-1"
        ECR_REGISTRY             = "453624448677.dkr.ecr.eu-north-1.amazonaws.com"
        ECR_REPOSITORY           = "jenkins"
        DEPLOY_SERVER_IP         = "16.170.237.29"
        IMAGE_TAG                = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                git branch: 'master', url: 'https://github.com/athirag30/cicd.git'
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Building and testing the application...'
                dir('app') {
                    sh 'npm install'
                    // If tests fail, continue anyway for now
                    sh 'npm test || echo "Tests completed with exit code: $?"'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_TAG}"
                dir('app') {
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                echo "Pushing Docker image to ECR..."
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                sh "docker push ${IMAGE_TAG}"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying application..."
                sh """
                    # Stop and remove old container if exists
                    docker stop webapp-container || true
                    docker rm webapp-container || true
                    
                    # Run new container
                    docker run -d -p 3000:3000 --name webapp-container ${IMAGE_TAG}
                    
                    echo "Deployment complete!"
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
            sh 'docker system prune -f'
        }
        success {
            echo 'SUCCESS: Pipeline completed successfully!'
            sh """
                echo "Application should be available at: http://${DEPLOY_SERVER_IP}:3000"
                echo "Current running containers:"
                docker ps
            """
        }
        failure {
            echo 'FAILED: Pipeline failed!'
        }
    }
}
