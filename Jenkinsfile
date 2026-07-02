pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "336359748215.dkr.ecr.ap-south-1.amazonaws.com/terraform-aws-eks"
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

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[
                   $class: 'AmazonWebServicesCredentialsBinding',
                   credentialsId: 'aws-credentials'
                ]]) {
                    sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl rollout status deployment/terraform-aws-eks -n dev

                    kubectl get deployments -n dev
                    kubectl get pods -n dev
                    kubectl get svc -n dev
                    '''
           }
         }
   }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Configure Kubernetes') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {

                    sh '''
                    aws eks update-kubeconfig \
                    --region ${AWS_REGION} \
                    --name ${CLUSTER_NAME}

                    kubectl config current-context
                    '''
                }
            }
        }

        stage('Verify Cluster') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {

                    sh '''
                    kubectl get nodes
                    kubectl get namespaces
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {

                    sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl rollout status deployment/terraform-aws-eks
                    kubectl get pods
                    kubectl get svc
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "=================================="
            echo "Pipeline completed successfully!"
            echo "Docker image pushed to ECR."
            echo "Application deployed to EKS."
            echo "=================================="
        }

        failure {
            echo "=================================="
            echo "Pipeline failed."
            echo "Check the stage logs above."
            echo "=================================="
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
