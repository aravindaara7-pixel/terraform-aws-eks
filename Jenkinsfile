pipeline {
    agent any

    environment {
        AWS_REGION  = 'ap-south-1'
        ACCOUNT_ID  = '336359748215'
        ECR_REPO    = '336359748215.dkr.ecr.ap-south-1.amazonaws.com/terraform-aws-eks'
        IMAGE_NAME  = 'terraform-aws-eks'
        IMAGE_TAG   = 'v1'
        CLUSTER_NAME = 'multi-env-eks'
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

        stage('AWS Login') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']
                ]) {
                    sh '''
                        aws sts get-caller-identity

                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login \
                        --username AWS \
                        --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
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
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']
                ]) {
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
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']
                ]) {
                    sh '''
                        kubectl get nodes
                        kubectl get namespaces
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials']
                ]) {
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
    }

    post {

        success {
            echo "======================================="
            echo "Pipeline completed successfully!"
            echo "Docker image pushed to Amazon ECR."
            echo "Application deployed to Amazon EKS."
            echo "======================================="
        }

        failure {
            echo "======================================="
            echo "Pipeline failed."
            echo "Please check the stage logs."
            echo "======================================="
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
