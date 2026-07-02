pipeline {
    agent any

    environment {
        AWS_REGION   = 'ap-south-1'
        ACCOUNT_ID   = '336359748215'
        ECR_REPO     = '336359748215.dkr.ecr.ap-south-1.amazonaws.com/terraform-aws-eks'
        IMAGE_NAME   = 'terraform-aws-eks'
        IMAGE_TAG    = "v1"
        CLUSTER_NAME = 'multi-env-eks'
        RELEASE_NAME = 'terraform-app'
        NAMESPACE    = 'dev'
        CHART_PATH   = './terraform-aws-eks-chart'
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
                sh '''
                    kubectl get nodes
                    kubectl get namespaces
                '''
            }
        }

        stage('Deploy using Helm') {
            steps {
                sh '''
                    helm version

                    helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                        --namespace ${NAMESPACE} \
                        --create-namespace \
                        --set image.repository=${ECR_REPO} \
                        --set image.tag=${IMAGE_TAG}

                    kubectl rollout status deployment/${IMAGE_NAME} -n ${NAMESPACE}

                    kubectl get pods -n ${NAMESPACE}
                    kubectl get svc -n ${NAMESPACE}
                    kubectl get ingress -n ${NAMESPACE}

                    helm list -n ${NAMESPACE}
                '''
            }
        }
    }

    post {

        success {
            echo "========================================="
            echo "Docker Image Built Successfully"
            echo "Image Pushed to Amazon ECR"
            echo "Application Deployed using Helm"
            echo "========================================="
        }

        failure {
            echo "========================================="
            echo "Pipeline Failed"
            echo "========================================="
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}
