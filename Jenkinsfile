pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        REGISTRY = "junga970"   // Docker Hub 계정명
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Images') {
            steps {
                script {
                    // 전체 서비스 리스트
                    def services = [
                        "chat-service",
                        "discovery-service",
                        "file-service",
                        "gateway-service",
                        "post-service",
                        "product-service",
                        "review-service",
                        "user-service",
                        "websocket-service"
                    ]

                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        for (service in services) {
                            dir(service) {
                                sh """
                                echo "Building Docker image for ${service}"
                                docker build -t ${REGISTRY}/${service}:dev-${env.BUILD_NUMBER} .
                                docker push ${REGISTRY}/${service}:dev-${env.BUILD_NUMBER}
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def services = [
                        "chat-service",
                        "discovery-service",
                        "file-service",
                        "gateway-service",
                        "post-service",
                        "product-service",
                        "review-service",
                        "user-service",
                        "websocket-service"
                    ]

                    for (service in services) {
                        sh """
                        echo "Deploying ${service} to Kubernetes..."
                        kubectl set image deployment/${service} ${service}=${REGISTRY}/${service}:dev-${env.BUILD_NUMBER} -n momnect
                        """
                    }
                }
            }
        }
    }
}
