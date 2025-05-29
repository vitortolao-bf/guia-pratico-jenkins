pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '343236792564.dkr.ecr.us-east-1.amazonaws.com/bf-jenkins'
        IMAGE_NAME = "bf-jenkins/guia-jenkins"
        IMAGE_TAG = "${BUILD_ID}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} -f ./src/Dockerfile ./src"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Clean Up Docker Image') {
            steps {
                script {
                    sh "docker rmi ${ECR_REPO}:${IMAGE_TAG} || true"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        sh """
                            kubectl set image deployment/bf-jenkins-deployment \
                            bf-jenkins-container=${ECR_REPO}:${IMAGE_TAG} \
                            --kubeconfig $KUBECONFIG
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deploy realizado com sucesso no EKS!"
        }
        failure {
            echo "❌ Erro no pipeline de deploy"
        }
    }
}
