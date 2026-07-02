pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REGISTRY = "336359748215.dkr.ecr.ap-south-1.amazonaws.com"
        ECR_REPOSITORY = "terraform-aws-eks"
        IMAGE_NAME = "terraform-aws-eks"
        IMAGE_TAG = "v1"
        CLUSTER_NAME = "multi-env-eks"
    }

    stages {

        stage('Clone Repository') {
           steps {
              git branch: 'master',
                url: 'https://github.com/aravindaara7-pixel/terraform-aws-eks.git'
          }
      }
  
        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image to Amazon ECR') {
            steps {
                sh '''
                docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Configure Kubernetes') {
            steps {
                sh '''
                aws eks update-kubeconfig \
                --region ${AWS_REGION} \
                --name ${CLUSTER_NAME}
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f ecr-deployment/namespace.yaml
                kubectl apply -f ecr-deployment/deployment.yaml
                kubectl apply -f ecr-deployment/service.yaml
                kubectl apply -f ecr-deployment/ingress.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get nodes
                kubectl get deployments -n dev
                kubectl get pods -n dev
                kubectl get svc -n dev
                kubectl get ingress -n dev
                '''
            }
        }
    }

    post {
        success {
            echo "========================================"
            echo "CI/CD Pipeline Executed Successfully"
            echo "Repository Cloned"
            echo "Docker Image Built"
            echo "Image Pushed to Amazon ECR"
            echo "Application Deployed to Amazon EKS"
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "CI/CD Pipeline Failed"
            echo "Please check the Jenkins Console Output"
            echo "========================================"
        }
    }
}
