pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '343236792564.dkr.ecr.us-east-1.amazonaws.com/bf-jenkins'
        IMAGE_NAME = "bf-jenkins/guia-jenkins"
        IMAGE_TAG = "${env.BUILD_ID}"
        KUBE_CONFIG = credentials('kubeconfig') // credencial armazenada no Jenkins
        // DOCKER_REGISTRY = "https://registry.hub.docker.com"
        // DOCKER_CREDENTIALS_ID = "dockerhub"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Cria a imagem e armazena na variável image
                    // image = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}", '-f ./src/Dockerfile ./src')
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} -f ./src/Dockerfile ./src"
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO
                    """
                    // aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 343236792564.dkr.ecr.us-east-1.amazonaws.com
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                    // Usa withRegistry com imagem criada
                    // docker.withRegistry("${env.DOCKER_REGISTRY}", "${env.DOCKER_CREDENTIALS_ID}") {
                    //     image.push("${env.BUILD_ID}")  // faz o push da imagem com a tag BUILD_ID
                    //     image.push('latest')          // opcional: também faz push com a tag 'latest'
                    }
                }
            }
        }

        stage('Clean Up Docker Image') {
            steps {
                script {
                    // Remove as imagens locais para liberar espaço
                    sh "docker rmi ${env.IMAGE_NAME}:${env.IMAGE_TAG} || true"
                    sh "docker rmi ${env.IMAGE_NAME}:latest || true"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        sh """
                            kubectl set image .k8s/deployment \
                            image=${ECR_REPO}:${IMAGE_TAG} \
                            --kubeconfig $KUBECONFIG
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deploy realizado com sucesso no EKS!"
        }
        failure {
            echo "Erro no pipeline de deploy"
        }
    }
}
