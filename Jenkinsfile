pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '343236792564.dkr.ecr.us-east-1.amazonaws.com/bf-jenkins'
        IMAGE_NAME = "guia-jenkins"
        IMAGE_TAG = "${BUILD_ID}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker build -t ${ECR_REPO}:${IMAGE_TAG} -f ./src/Dockerfile ./src
                    '''
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh '''
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Clean Up Docker Image') {
            steps {
                script {
                    sh '''
                        docker rmi ${ECR_REPO}:${IMAGE_TAG} || true
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                        script {
                            sh '''
                                kubectl set image ./k8s/deployment \
                                conversao-temperatura=${ECR_REPO}:${IMAGE_TAG}
                                kubectl apply -f ./k8s/deployment
                            '''
                        }
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
