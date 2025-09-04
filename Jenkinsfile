pipeline {
    agent any
    triggers {
        githubPush()   // GitHub Webhook 트리거
    }
    environment {
        REGISTRY = "docker.io/junga970"  // Docker Hub 계정
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changed Services') {
            steps {
                script {
                    // dev 브랜치 기준 변경된 파일 확인
                    def changedFiles = sh(
                        script: "git diff --name-only origin/dev",
                        returnStdout: true
                    ).trim().split("\n")

                    def services = []
                    for (file in changedFiles) {
                        if (file.startsWith("chat-service/")) services << "chat-service"
                        if (file.startsWith("discovery-service/")) services << "discovery-service"
                        if (file.startsWith("file-service/")) services << "file-service"
                        if (file.startsWith("gateway-service/")) services << "gateway-service"
                        if (file.startsWith("post-service/")) services << "post-service"
                        if (file.startsWith("product-service/")) services << "product-service"
                        if (file.startsWith("review-service/")) services << "review-service"
                        if (file.startsWith("user-service/")) services << "user-service"
                        if (file.startsWith("websocket-service/")) services << "websocket-service"
                    }
                    env.CHANGED_SERVICES = services.unique().join(" ")
                    echo "Changed Services: ${env.CHANGED_SERVICES}"
                }
            }
        }

        stage('Build & Push Docker Images') {
            when { expression { return env.CHANGED_SERVICES?.trim() } }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        for (service in env.CHANGED_SERVICES.split(" ")) {
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
            when { expression { return env.CHANGED_SERVICES?.trim() } }
            steps {
                script {
                    for (service in env.CHANGED_SERVICES.split(" ")) {
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
