pipeline {
    agent any

    environment {
        IMAGE_NAME = "devopsbemfacil/guia-jenkins"
        DOCKER_REGISTRY = "https://registry.hub.docker.com"
        DOCKER_CREDENTIALS_ID = "dockerhub"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Cria a imagem e armazena na variável image
                    image = docker.build("${env.IMAGE_NAME}:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Usa withRegistry com imagem criada
                    docker.withRegistry("${env.DOCKER_REGISTRY}", "${env.DOCKER_CREDENTIALS_ID}") {
                        image.push("${env.BUILD_ID}")  // faz o push da imagem com a tag BUILD_ID
                        image.push('latest')          // opcional: também faz push com a tag 'latest'
                    }
                }
            }
        }

        stage('Clean Up Docker Image') {
            steps {
                script {
                    // Remove as imagens locais para liberar espaço
                    sh "docker rmi ${env.IMAGE_NAME}:${env.BUILD_ID} || true"
                    sh "docker rmi ${env.IMAGE_NAME}:latest || true"
                }
            }
        }

        stage('Deploy no Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}
