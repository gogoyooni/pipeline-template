// Jenkinsfile
pipeline {
    agent {
        kubernetes {
           cloud 'kubernetes'
            label 'kubeagent'  // Jenkins UI의 Pod Template label과 일치
            inheritFrom 'kube-agent'  // Jenkins UI의 Pod Template name과 일치
            yaml """
                ${params.POD_TEMPLATE}
            """
        }
    }
    stages {
        stage('Checkout') {
            steps {
                git url: params.GIT_REPO
            }
        }
        stage('Build') {
            steps {
                container('kaniko') {
                    // Kaniko를 사용하여 Docker 이미지 빌드 및 푸시
                    sh """
                        /kaniko/executor \
                            --context=. \
                            --dockerfile=Dockerfile \
                            --destination=${params.DOCKER_USERNAME}/${params.PROJECT_NAME}:v${env.BUILD_NUMBER}
                    """
                }
            }
        }
        stage('Deploy') {
            steps {
                container('kubectl') {
                    // Kubernetes 매니페스트 적용
                    sh """
                        kubectl apply -f k8s/deployment.yaml -n ${params.NAMESPACE}
                    """
                }
            }
        }
    }
}
