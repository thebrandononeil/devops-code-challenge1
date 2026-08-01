pipeline {
    agent any

    environment {
        AWS_REGION       = 'us-east-1'
        AWS_ACCOUNT_ID   = '294790491016'
        ECR_REGISTRY     = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        BACKEND_REPO     = 'tech-challenge-backend'
        FRONTEND_REPO    = 'tech-challenge-frontend'

        ECS_CLUSTER      = 'tech-challenge-cluster'
        BACKEND_SERVICE  = 'tech-challenge-backend-service'
        FRONTEND_SERVICE = 'tech-challenge-frontend-service'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    git --version
                    docker --version
                    aws --version
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region "$AWS_REGION" | docker login \
                      --username AWS \
                      --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build \
                      -t "$BACKEND_REPO:$BUILD_NUMBER" \
                      ./backend

                    docker build \
                      -t "$FRONTEND_REPO:$BUILD_NUMBER" \
                      ./frontend
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                    docker tag \
                      "$BACKEND_REPO:$BUILD_NUMBER" \
                      "$ECR_REGISTRY/$BACKEND_REPO:$BUILD_NUMBER"

                    docker tag \
                      "$BACKEND_REPO:$BUILD_NUMBER" \
                      "$ECR_REGISTRY/$BACKEND_REPO:latest"

                    docker tag \
                      "$FRONTEND_REPO:$BUILD_NUMBER" \
                      "$ECR_REGISTRY/$FRONTEND_REPO:$BUILD_NUMBER"

                    docker tag \
                      "$FRONTEND_REPO:$BUILD_NUMBER" \
                      "$ECR_REGISTRY/$FRONTEND_REPO:latest"
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    docker push "$ECR_REGISTRY/$BACKEND_REPO:$BUILD_NUMBER"
                    docker push "$ECR_REGISTRY/$BACKEND_REPO:latest"

                    docker push "$ECR_REGISTRY/$FRONTEND_REPO:$BUILD_NUMBER"
                    docker push "$ECR_REGISTRY/$FRONTEND_REPO:latest"
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                    aws ecs update-service \
                      --cluster "$ECS_CLUSTER" \
                      --service "$BACKEND_SERVICE" \
                      --force-new-deployment \
                      --region "$AWS_REGION"

                    aws ecs update-service \
                      --cluster "$ECS_CLUSTER" \
                      --service "$FRONTEND_SERVICE" \
                      --force-new-deployment \
                      --region "$AWS_REGION"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage in Console Output.'
        }

        always {
            sh 'docker system prune -af || true'
        }
    }
}