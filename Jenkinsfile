pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER_NAME = "managed-eks-cluster"

        // Jenkins credentials
        DOCKERHUB_CREDS = credentials('dockerhub-creds')

        // AWS credentials
        AWS_CREDS = credentials('aws-creds')

        AWS_ACCESS_KEY_ID     = "${AWS_CREDS_USR}"
        AWS_SECRET_ACCESS_KEY = "${AWS_CREDS_PSW}"

        // Image details
        IMAGE_NAME = "${DOCKERHUB_CREDS_USR}/myapp"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/umasankarbgit/project-6.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t $FULL_IMAGE .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh '''
                    echo "$DOCKERHUB_CREDS_PSW" | \
                    docker login -u "$DOCKERHUB_CREDS_USR" --password-stdin
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                    echo "Pushing image..."
                    docker push $FULL_IMAGE
                '''
            }
        }

        stage('Configure AWS CLI') {
            steps {
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_REGION
                '''
            }
        }

        stage('Update Kubeconfig') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                    --region $AWS_REGION \
                    --name $EKS_CLUSTER_NAME
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    sed -i "s|image:.*|image: $FULL_IMAGE|g" K8s/deployment.yaml

                    kubectl apply -f K8s/deployment.yaml
                    kubectl apply -f K8s/service.yaml

                    kubectl rollout status deployment/dockerhub-sample-app
                '''
            }
        }
    }

    post {
    success {
        echo "CI/CD pipeline completed successfully. App deployed to EKS!"
    }
    failure {
        echo "Pipeline failed. Please check logs."
    }
    always {
        sh 'docker logout'
    }
}
}
