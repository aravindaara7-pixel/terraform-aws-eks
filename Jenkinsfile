pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        ECR_REGISTRY   = "336359748215.dkr.ecr.ap-south-1.amazonaws.com"
        ECR_REPOSITORY = "terraform-aws-eks"
        IMAGE_TAG      = "v1"
        CLUSTER_NAME   = "multi-env-eks"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git 'https://github.com/aravindaara7-pixel/terraform-aws-eks.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t terraform-aws-eks:v1 .'
            }
        }

        stage('AWS Login') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']
                ]) {

                    sh '''
                    aws sts get-caller-identity

                    aws ecr get-login-password --region ap-south-1 | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    336359748215.dkr.ecr.ap-south-1.amazonaws.com
                    '''
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag terraform-aws-eks:v1 \
                336359748215.dkr.ecr.ap-south-1.amazonaws.com/terraform-aws-eks:v1
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push \
                336359748215.dkr.ecr.ap-south-1.amazonaws.com/terraform-aws-eks:v1
                '''
            }
        }

        stage('Configure Kubernetes') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']
                ]) {

                    sh '''
                    aws eks update-kubeconfig \
                    --region ap-south-1 \
                    --name multi-env-eks
                    '''
                }
            }
        }

        stage('Verify Cluster') {
            steps {
                sh '''
                kubectl get nodes
                '''
            }
        }
    }
}
