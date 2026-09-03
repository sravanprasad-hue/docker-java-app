pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '226123187183.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'docker-instructions'
        IMAGE_NAME = 'docker-java-app'
        IMAGE_TAG = '1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sravanprasad-hue/docker-java-app.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Checking project files..."
                    ls -la
                    test -f Dockerfile
                    test -f pom.xml
                    test -d src
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login \
                      --username AWS \
                      --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    docker tag \
                      ${IMAGE_NAME}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Verify ECR Image') {
            steps {
                sh '''
                    aws ecr describe-images \
                      --repository-name ${ECR_REPOSITORY} \
                      --image-ids imageTag=${IMAGE_TAG} \
                      --region ${AWS_REGION}
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker image successfully built and pushed to Amazon ECR!'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage in Jenkins.'
        }
    }
}
        stage('Deploy Container') {
            steps {
                sh '''
                    echo "Stopping existing container if present..."
                    docker rm -f docker-java-app-container || true

                    echo "Pulling image from ECR..."
                    docker pull ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    echo "Starting new container..."
                    docker run -d \
                      --name docker-java-app-container \
                      -p 8080:8080 \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    echo "Checking running container..."
                    docker ps

                    echo "Testing application..."
                    sleep 5
                    curl -f http://localhost:8080 || exit 1
                '''
            }
        }
