pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes'
            yaml """
                kind: Pod
                metadata:
                  namespace: ${params.NAMESPACE}
                  labels:
                    kubeagent: "true"
                spec:
                  serviceAccountName: ${params.NAMESPACE}-sa
                  containers:
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:debug
                    command:
                    - /busybox/cat
                    tty: true
                    volumeMounts:
                    - name: docker-config
                      mountPath: /kaniko/.docker/
                  - name: kubectl
                    image: bitnami/kubectl:latest
                    command:
                    - sleep
                    args:
                    - 99d
                    tty: true
                  volumes:
                  - name: docker-config
                    secret:
                      secretName: docker-credentials
                      items:
                        - key: .dockerconfigjson
                          path: config.json
            """
            defaultContainer 'kubectl'
        }
    }
    
    environment {
        DOCKER_IMAGE = "${params.DOCKER_USERNAME}/dynamic-jenkins-test"
        DOCKER_TAG = "${BUILD_NUMBER}"
        APP_NAME = "line-app"
    }
    
    stages {
        stage('Print Parameters') {
            steps {
                echo "Namespace: ${params.NAMESPACE}"
                echo "Project Name: ${params.PROJECT_NAME}"
                echo "Docker Username: ${params.DOCKER_USERNAME}"
                echo "Git Repo: ${params.GIT_REPO}"
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                container('kaniko') {
                    sh """
                        # 현재 디렉토리 내용 확인
                        ls -la
                
                        # package.json 파일 찾기
                        find . -name "package.json"
                        /kaniko/executor \
                        --context=\$PWD \
                        --dockerfile=\$PWD/Dockerfile \
                        --destination=${DOCKER_IMAGE}:${DOCKER_TAG}
                       
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh """
                        # 기존 deployment 삭제
                        kubectl delete deployment ${APP_NAME} -n ${params.NAMESPACE} --ignore-not-found=true
                        
                        # deployment 생성
                        cat <<EOF | kubectl apply -f -
                        apiVersion: apps/v1
                        kind: Deployment
                        metadata:
                          name: ${APP_NAME}
                          namespace: ${params.NAMESPACE}
                        spec:
                          replicas: 1
                          selector:
                            matchLabels:
                              app: ${APP_NAME}
                          template:
                            metadata:
                              labels:
                                app: ${APP_NAME}
                            spec:
                              containers:
                              - name: ${APP_NAME}
                                image: ${DOCKER_IMAGE}:${DOCKER_TAG}
                                ports:
                                - containerPort: 3000
                        EOF
                        
                        # deployment 상태 확인
                        kubectl rollout status deployment/${APP_NAME} -n ${params.NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
