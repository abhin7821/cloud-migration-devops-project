pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '442122590814'
        REGION = 'us-east-1'
        ECR_REPO = 'cloud-migration-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Building Docker image for $ECR_REPO:$IMAGE_TAG ..."
                    sh 'docker build -t $ECR_REPO:$IMAGE_TAG ./app'
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withAWS(credentials: 'aws-jenkins-creds', region: "${REGION}") {
                    script {
                        echo "🔑 Logging in to AWS ECR..."
                        sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com'
                    }
                }
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                script {
                    echo "🚀 Tagging and pushing Docker image to ECR..."
                    sh '''
                        docker tag $ECR_REPO:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                        docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-jenkins-creds', region: "${REGION}") {
                    script {
                        echo "🚀 Deploying image to EKS cluster..."
                        sh '''
                            aws eks update-kubeconfig --region $REGION --name cloud-migration-eks
                            kubectl set image deployment/cloud-migration-app \
                              cloud-migration-app=$AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG \
                              -n cloud-migration
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD pipeline completed successfully! Image tag: $IMAGE_TAG"
        }
        failure {
            echo "❌ Pipeline failed! Check logs in Jenkins console output."
        }
    }
}
