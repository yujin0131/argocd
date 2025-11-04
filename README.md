# ArgoCD App of Apps 프로젝트
이 프로젝트는 ArgoCD의 App of Apps 패턴을 사용하여 헬름 차트 기반 애플리케이션을 관리합니다.

## 프로젝트 구조

```
argocd/
├── Chart.yaml
├── external-secrets.yaml
├── policies/
│   ├── values.csv
├── templates/
│   ├── argocd-apps.yaml
│   ├── configmaps.yaml
│   ├── namespaces.yaml
│   ├── repositories.yaml
│   └── secrets.yaml
├── values/
│   ├── app-projects.yaml
│   ├── repository.yaml
│   └── values.disclosure.yaml
├── docs/
└── README.md
```

**참고**: 다른 구조 예시는 [docs 문서](docs/STRUCTURE_OPTIONS.md)를 참조하세요.

## 구성 요소

### 1. argocd 설치
```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm install argocd argo/argo-cd \
  -n argocd --create-namespace \
  -f ./values.yaml \
  -f ./values/repository.yaml \
  -f ./values/app-projects.yaml \
  -f ./values/values.disclosure.yaml
```

### 2. 주요 리소스 설명
- **repositories**: Helm 또는 Git 리포지토리 정의
- **projects**: ArgoCD 프로젝트 정의
- **policies**: RBAC 정책 및 접근 제어 설정
- **secrets**: 인증 및 시크릿 관리

### 3. Values 구조
- 각 애플리케이션별 Helm values 파일 관리
- 환경(dev, prod 등)별 설정 파일 분리 가능

## 사용 방법

### 1. 초기 설정
```bash
# ArgoCD에 루트 애플리케이션 생성
kubectl apply -f root-app.yaml
```

### 2. Values 수정
각 애플리케이션의 values 파일을 환경에 맞게 수정:
- `values.yaml`
- `values/repository.yaml`
- `values/app-projects.yaml`
- `values/policies/values.csv`
- `values/secrets.yaml`

### 3. ArgoCD에서 확인
ArgoCD UI 또는 CLI를 통해 애플리케이션 상태 확인:
```bash
argocd app list
```

## 헬름 차트 설정

각 애플리케이션은 헬름 차트를 사용합니다. 필요한 헬름 차트 정보를 `applications/*/app.yaml` 파일에 설정하세요.

## 주의사항

1. **Secrets 관리**: 민감한 정보는 Sealed Secrets, External Secrets Operator 등과 함께 사용하는 것을 권장합니다.
2. **Repository URL**: 헬름 리포지토리 URL은 실제 사용하는 리포지토리로 변경해야 합니다.
3. **Namespace**: 각 애플리케이션의 네임스페이스를 환경에 맞게 설정하세요.

## 커스터마이징

환경별(values-dev.yaml, values-prod.yaml) 또는 클러스터별 설정이 필요한 경우, 각 애플리케이션의 values 파일을 분리하여 관리할 수 있습니다.
