pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'my-sample-app'
        ECR_URI = "114593495110.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out latest code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${ECR_REPO}:latest .'
            }
        }

        stage('Push to ECR') {
            steps {
                echo 'Pushing Docker image to AWS ECR...'
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        set -e
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
                        
                        # Get ECR login
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin ${ECR_URI%/*}
                        
                        # Tag and push image
                        docker tag ${ECR_REPO}:latest $ECR_URI:latest
                        docker push $ECR_URI:latest
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                echo "Deploying container on EC2..."
                sh '''
                # Stop and remove old container if it exists
                if [ "$(docker ps -aq -f name=my-sample-app)" ]; then
                echo "🧹 Removing old container..."
                docker stop my-sample-app || true
                docker rm my-sample-app || true
            fi

        # Run new container
        docker run -d -p 80:80 --name my-sample-app 114593495110.dkr.ecr.us-east-1.amazonaws.com/my-sample-app:latest
        '''
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
}
