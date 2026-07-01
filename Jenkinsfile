pipeline {

    agent any

    environment {
        DEVOPS_SERVER = "ubuntu@172.31.8.83"
        PROJECT_DIR   = "/home/ubuntu/eks-multi-env"

        AWS_REGION    = "ap-south-1"
        CLUSTER_NAME  = "multi-env-eks"

        ACCOUNT_ID    = "336359748215"
        IMAGE_NAME    = "terraform-aws-eks"
        IMAGE_TAG     = "v1"

        ECR_REPO      = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_NAME}"

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
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            echo "===== SSH Connected ====="
                            hostname
                            whoami
                            pwd
                        '
                    """
                }
            }
        }

        stage('Create Project Directory') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            mkdir -p ${PROJECT_DIR}
                        '
                    """
                }
            }
        }

        stage('Copy Project Files') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        rsync -av --delete \
                        -e "ssh -o StrictHostKeyChecking=no" \
                        ./ ${DEVOPS_SERVER}:${PROJECT_DIR}/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            cd ${PROJECT_DIR}

                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        '
                    """
                }
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login \
                            --username AWS \
                            --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        '
                    """
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                        '
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            docker push ${ECR_REPO}:${IMAGE_TAG}
                        '
                    """
                }
            }
        }

        stage('Configure EKS') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            aws eks update-kubeconfig \
                                --region ${AWS_REGION} \
                                --name ${CLUSTER_NAME}
                        '
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            cd ${PROJECT_DIR}

                            kubectl apply -f namespace.yaml
                            kubectl apply -f deployment.yaml
                            kubectl apply -f service.yaml
                            kubectl apply -f ingress.yaml
                        '
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['devops-ec2']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEVOPS_SERVER} '
                            echo "===== Pods ====="
                            kubectl get pods -n ${NAMESPACE}

                            echo "===== Services ====="
                            kubectl get svc -n ${NAMESPACE}

                            echo "===== Ingress ====="
                            kubectl get ingress -n ${NAMESPACE}
                        '
                    """
                }
            }
        }
    }

    post {

        success {
            echo "========================================"
            echo " CI/CD PIPELINE COMPLETED SUCCESSFULLY "
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo " CI/CD PIPELINE FAILED "
            echo "========================================"
        }

        always {
            cleanWs()
        }
    }
}
