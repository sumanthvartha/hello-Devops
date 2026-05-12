pipeline {
    agent any
    
    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '237612937960.dkr.ecr.ap-south-1.amazonaws.com/hello-devops'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        CLUSTER_NAME = 'hello-devops-cluster'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }
        
        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                        kubectl set image deployment/hello-devops hello-devops=${ECR_REPO}:${IMAGE_TAG} || \
                        kubectl create deployment hello-devops --image=${ECR_REPO}:${IMAGE_TAG}
                        kubectl expose deployment hello-devops --type=LoadBalancer --port=5000 || true
                    """
                }
            }
        }
    }
}