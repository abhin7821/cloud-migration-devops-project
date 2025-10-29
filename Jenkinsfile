pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '442122590814'
        REGION = 'us-east-1'
        ECR_REPO = 'cloud-migration-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/abhin7821/cloud-migration-devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $ECR_REPO:$IMAGE_TAG ./app'
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com'
                }
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                script {
                    sh '''
                    docker tag $ECR_REPO:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                    docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }
    }
}
