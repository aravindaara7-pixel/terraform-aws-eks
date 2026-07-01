pipeline {
 
    agent any
 
    environment {
        DEVOPS_SERVER = "ubuntu@172.31.8.83"
        PROJECT_DIR   = "/home/ubuntu/eks-multi-env"
 
        AWS_REGION    = "ap-south-1"
        CLUSTER_NAME  = "multi-env-eks"
 
        ACCOUNT_ID    = "336359748215"
        ECR_REPO      = "336359748215.dkr.ecr.ap-south-1.amazonaws.com/terraform-aws-eks"
 
        IMAGE_NAME    = "terraform-aws-eks"
        IMAGE_TAG     = "v1"
 
        NAMESPACE     = "dev"
    }
 
    stages {
 
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }
 
        stage('Verify SSH Connection') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    hostname
                    pwd
                    "
                    '''
                }
            }
        }
 
        stage('Copy Project Files') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    rsync -av --delete -e "ssh -o StrictHostKeyChecking=no" ./ $DEVOPS_SERVER:$PROJECT_DIR/
                    '''
                }
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    cd $PROJECT_DIR
 
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    "
                    '''
                }
            }
        }
 
        stage('Login to Amazon ECR') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    "
                    '''
                }
            }
        }
 
        stage('Tag Docker Image') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    docker tag $IMAGE_NAME:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG
                    "
                    '''
                }
            }
        }
 
        stage('Push Docker Image') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    docker push $ECR_REPO:$IMAGE_TAG
                    "
                    '''
                }
            }
        }
 
        stage('Configure EKS') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                    "
                    '''
                }
            }
        }
 
        stage('Deploy Application') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    cd $PROJECT_DIR
 
                    kubectl apply -f namespace.yaml
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    kubectl apply -f ingress.yaml
                    "
                    '''
                }
            }
        }
 
        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEVOPS_SERVER "
                    kubectl get pods -n $NAMESPACE
                    kubectl get svc -n $NAMESPACE
                    kubectl get ingress -n $NAMESPACE
                    "
                    '''
                }
            }
        }
 
    }
 
    post {
 
        success {
            echo "======================================"
            echo " CI/CD PIPELINE COMPLETED SUCCESSFULLY "
            echo "======================================"
        }
 
        failure {
            echo "======================================"
            echo " CI/CD PIPELINE FAILED "
            echo "======================================"
        }
 
        always {
            cleanWs()
        }
 
    }
 
}
