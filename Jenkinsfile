pipeline {
    agent any

    environment {
        ACCOUNT_ID       = "489364174421"
        AWS_REGION       = "ap-south-1"

        FRONTEND_REPO    = "mern-frontend"
        BACKEND_REPO     = "mern-backend"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/azuredataengineer555-debug/mern-ecommerce-practice.git'
            }
        }

        stage('Build Frontend App') {
            agent { label 'deploy' }
            steps {
                dir('MERN-Stack-Ecommerce-App-master') {
                    sh '''
                        echo "=== BUILDING FRONTEND ==="
                        npm install
                        CI=false npm run build
                    '''
                }
            }
        }

        stage('Build Backend App') {
            agent { label 'deploy' }
            steps {
                dir('MERN-Stack-Ecommerce-App-master/backend') {
                    sh '''
                        echo "=== INSTALLING BACKEND DEPENDENCIES ==="
                        npm install
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            agent { label 'deploy' }
            steps {
                sh '''
                    echo "=== BUILDING FRONTEND IMAGE ==="
                    docker build -t ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:latest \
                    -f MERN-Stack-Ecommerce-App-master/Dockerfile MERN-Stack-Ecommerce-App-master

                    echo "=== BUILDING BACKEND IMAGE ==="
                    docker build -t ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:latest \
                    -f MERN-Stack-Ecommerce-App-master/backend/Dockerfile MERN-Stack-Ecommerce-App-master/backend
                '''
            }
        }

        stage('Login to AWS ECR') {
            agent { label 'deploy' }
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Push Images to ECR') {
            agent { label 'deploy' }
            steps {
                sh '''
                    docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:latest
                    docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:latest
                '''
            }
        }

        stage('Deploy Containers') {
            agent { label 'deploy' }
            steps {
                sh '''
                    docker rm -f frontend || true
                    docker rm -f backend || true

                    docker run -d --name frontend -p 80:80 ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:latest
                    docker run -d --name backend -p 5000:5000 ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed — Check logs"
        }
    }
}
