pipeline {
    agent any
    environment {
        REGISTRY = "junga970"
        NAMESPACE = "momnect"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Initial Resources') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'KUBECONFIG_EC2', variable: 'KUBECONFIG')]) {
                        // 네임스페이스 없으면 생성
                        sh "kubectl --kubeconfig=$KUBECONFIG get ns ${NAMESPACE} || kubectl --kubeconfig=$KUBECONFIG create ns ${NAMESPACE}"

                        // 최초 1회만 전체 YAML 반영
                        sh "kubectl --kubeconfig=$KUBECONFIG apply -f z-k8s/ -n ${NAMESPACE}"
                    }
                }
            }
        }
    }
}
