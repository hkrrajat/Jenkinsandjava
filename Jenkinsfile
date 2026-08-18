pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPOSITORY = 'jenkinsecr'
        IMAGE_NAME = 'helloworld'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/hkrrajat/Jenkinsandjava.git'
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    echo "Java version:"
                    java -version

                    echo "Maven version:"
                    mvn -version

                    echo "Docker version:"
                    docker --version

                    echo "AWS CLI version:"
                    aws --version
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    mvn clean package
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
                        --query Account \
                        --output text)

                    aws ecr get-login-password \
                        --region ${AWS_REGION} | \
                    docker login \
                        --username AWS \
                        --password-stdin \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
                        --query Account \
                        --output text)

                    docker tag \
                        ${IMAGE_NAME}:${BUILD_NUMBER} \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
                        --query Account \
                        --output text)

                    docker push \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            echo 'Application successfully built and pushed to ECR.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}