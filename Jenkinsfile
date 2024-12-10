// Jenkinsfile
pipeline {
    agent {
        kubernetes {
            yaml params.POD_TEMPLATE  // jenkins.service.ts의 getPodTemplate에서 정의한 Pod 스펙
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
