# Kiwi Platform Pipeline Template

이 저장소는 Kiwi Platform의 CI/CD 파이프라인 템플릿을 포함하고 있습니다.

## 파이프라인 구조

Jenkins 파이프라인은 다음 단계들로 구성되어 있습니다:

1. **Checkout**: 사용자의 GitHub 저장소에서 애플리케이션 코드를 가져옵니다.
2. **Build**: Kaniko를 사용하여 Docker 이미지를 빌드하고 Docker Hub에 푸시합니다.
3. **Deploy**: kubectl을 사용하여 Kubernetes 클러스터에 애플리케이션을 배포합니다.

## 필수 파라미터

파이프라인은 다음 파라미터들을 필요로 합니다:

- `NAMESPACE`: Kubernetes 네임스페이스
- `PROJECT_NAME`: 프로젝트 이름
- `DOCKER_USERNAME`: Docker Hub 사용자명
- `GIT_REPO`: 애플리케이션 소스 코드가 있는 GitHub 저장소 URL

## Pod 템플릿

파이프라인은 다음 컨테이너들을 포함하는 Kubernetes Pod에서 실행됩니다:

- **kaniko**: Docker 이미지 빌드 및 푸시
- **kubectl**: Kubernetes 배포

## 사용 방법

1. 애플리케이션 저장소에 다음 파일들이 필요합니다:
   - `Dockerfile`: 애플리케이션 빌드를 위한 Docker 설정
   - `k8s/deployment.yaml`: Kubernetes Deployment 매니페스트
   - `k8s/service.yaml`: Kubernetes Service 매니페스트

2. 이미지 태그는 Jenkins 빌드 번호를 사용합니다:
   - 형식: `${DOCKER_USERNAME}/${PROJECT_NAME}:${BUILD_NUMBER}`

## 주의사항

- Docker 자격증명은 `docker-credentials` Secret으로 제공되어야 합니다.
- Kubernetes 권한은 네임스페이스별 ServiceAccount를 통해 관리됩니다.

## 예시 매니페스트

### deployment.yaml
