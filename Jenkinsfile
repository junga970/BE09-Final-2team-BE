pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        REGISTRY = "junga970"       // Docker Hub 계정명
        NAMESPACE = "momnect"       // 쿠버네티스 네임스페이스
        PATH = "/usr/local/bin:/usr/bin:/bin"
    }

    stages {
        stage('Check Docker & Kubectl') {
            steps {
                sh '''
                echo "== Who am I? =="
                whoami
                echo "== Docker version =="
                docker --version || echo "docker not installed"
                echo "== Kubectl version =="
                kubectl version --client || echo "kubectl not installed"
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Images') {
            steps {
                script {
                    def services = [
                        "chat-service", "discovery-service", "file-service",
                        "gateway-service", "post-service", "product-service",
                        "review-service", "user-service", "websocket-service"
                    ]

                    withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        """

                        for (service in services) {
                            dir(service) {
                                sh """
                                echo "🚀 Building Docker image for ${service}"
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
                    withCredentials([file(credentialsId: 'KUBECONFIG_EC2', variable: 'KUBECONFIG')]) {
                        // 네임스페이스 보장
                        sh "kubectl --kubeconfig=$KUBECONFIG get ns ${NAMESPACE} || kubectl --kubeconfig=$KUBECONFIG create ns ${NAMESPACE}"

                        // YAML 반영
                        sh "kubectl --kubeconfig=$KUBECONFIG apply -f z-k8s/ -n ${NAMESPACE}"

                        // 롤링 업데이트
                        def services = [
                            "chat-service", "discovery-service", "file-service",
                            "gateway-service", "post-service", "product-service",
                            "review-service", "user-service", "websocket-service"
                        ]
                        for (service in services) {
                            sh """
                            kubectl --kubeconfig=$KUBECONFIG set image deployment/${service} ${service}=${REGISTRY}/${service}:dev-${env.BUILD_NUMBER} -n ${NAMESPACE} || true
                            """
                        }
                    }
                }
            }
        }
    }
}
