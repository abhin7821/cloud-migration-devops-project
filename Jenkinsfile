pipeline {
    agent any

    environment {
        // === AWS and ECR configuration ===
        AWS_ACCOUNT_ID = '442122590814'
        REGION = 'us-east-1'
        ECR_REPO = 'cloud-migration-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        // === Stage 1: Build Docker Image ===
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image for $ECR_REPO:$IMAGE_TAG ..."
                    sh 'docker build -t $ECR_REPO:$IMAGE_TAG ./app'
                }
            }
        }

        // === Stage 2: Login to AWS ECR ===
        stage('Login to AWS ECR') {
            steps {
                script {
                    echo "Logging in to AWS ECR..."
                    sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com'
                }
            }
        }

        // === Stage 3: Tag and Push to ECR ===
        stage('Tag & Push to ECR') {
            steps {
                script {
                    echo "Tagging and pushing image to ECR..."
                    sh '''
                        docker tag $ECR_REPO:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                        docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    // === Post Actions ===
    post {
        success {
            echo "✅ Build and push successful! Image tag: $IMAGE_TAG"
        }
        failure {
            echo "❌ Build failed! Please check logs."
        }
    }
}
