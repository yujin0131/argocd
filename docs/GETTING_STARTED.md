# ArgoCD App of Apps 시작 가이드

## 사전 요구사항

1. Kubernetes 클러스터 접근 권한
2. ArgoCD 설치 완료
3. kubectl 및 argocd CLI 설치

## 1단계: 프로젝트 설정

### 1.1 Git 리포지토리 설정

`root-app.yaml` 파일에서 리포지토리 URL을 실제 Git 리포지토리로 변경:

```yaml
source:
  repoURL: https://github.com/YOUR_ORG/argocd  # 변경 필요
  targetRevision: HEAD
  path: applications
```

### 1.2 헬름 차트 리포지토리 설정

각 애플리케이션 파일(`applications/*/*.yaml`)에서 헬름 차트 정보를 설정:

```yaml
source:
  repoURL: https://your-helm-repo.com  # 변경 필요
  chart: your-chart-name  # 변경 필요
  targetRevision: 1.0.0  # 변경 필요
```

### 1.3 Values 파일 커스터마이징

각 컴포넌트의 values 파일을 환경에 맞게 수정:

- `values/repositories/values.yaml`: 헬름 리포지토리 추가
- `values/projects/values.yaml`: ArgoCD 프로젝트 설정
- `values/policies/values.yaml`: 정책 설정
- `values/secrets/values.yaml`: 시크릿 참조 설정 (실제 비밀은 저장하지 않음)

## 2단계: ArgoCD에 배포

### 2.1 루트 애플리케이션 생성

```bash
kubectl apply -f root-app.yaml
```

### 2.2 ArgoCD CLI로 확인

```bash
# ArgoCD 로그인
argocd login argocd.example.com

# 애플리케이션 목록 확인
argocd app list

# 루트 애플리케이션 상태 확인
argocd app get root-app
```

### 2.3 애플리케이션 동기화

```bash
# 수동 동기화
argocd app sync root-app

# 자동 동기화가 설정되어 있으면 자동으로 동기화됩니다
```

## 3단계: 컴포넌트별 관리

### 3.1 Repository 관리

1. `values/repositories/values.yaml`에 리포지토리 추가
2. Git에 커밋 및 푸시
3. ArgoCD가 자동으로 동기화 (자동 동기화 설정 시)

또는 ArgoCD CLI로 직접 추가:

```bash
argocd repo add https://charts.bitnami.com/bitnami --type helm --name bitnami
```

### 3.2 Project 관리

1. `values/projects/values.yaml`에 프로젝트 정의 추가
2. Git에 커밋 및 푸시
3. ArgoCD가 프로젝트를 생성/업데이트

### 3.3 Policy 관리

1. `values/policies/values.yaml`에서 정책 수정
2. Git에 커밋 및 푸시
3. ArgoCD ConfigMap이 자동으로 업데이트

### 3.4 Secrets 관리

**주의**: 민감한 정보는 절대 Git에 커밋하지 마세요.

**방법 1: Sealed Secrets 사용 (권장)**

```bash
# Sealed Secrets Controller 설치
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Secret 생성 및 암호화
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret.yaml

# Git에 커밋 (안전)
git add sealed-secret.yaml
git commit -m "Add sealed secret"
```

**방법 2: External Secrets Operator 사용**

AWS Secrets Manager, HashiCorp Vault 등과 통합하여 시크릿을 관리합니다.

**방법 3: ArgoCD CLI 사용**

```bash
# Repository secret 추가
argocd repo add https://github.com/user/repo \
  --username git-user \
  --password git-password

# Cluster secret 추가
argocd cluster add https://kubernetes.example.com \
  --name production
```

## 4단계: 모니터링 및 유지보수

### 4.1 애플리케이션 상태 모니터링

```bash
# 모든 애플리케이션 상태 확인
argocd app list

# 특정 애플리케이션 상세 정보
argocd app get repositories

# 애플리케이션 리소스 트리 확인
argocd app resources repositories
```

### 4.2 문제 해결

```bash
# 애플리케이션 이벤트 확인
argocd app history repositories

# 로그 확인
argocd app logs repositories

# 수동 동기화 (문제 발생 시)
argocd app sync repositories --prune
```

## 5단계: 고급 설정

### 5.1 환경별 Values 분리

환경별 values 파일을 생성하여 관리:

```
values/
├── repositories/
│   ├── values-dev.yaml
│   ├── values-staging.yaml
│   └── values-prod.yaml
```

애플리케이션에서 여러 values 파일 참조:

```yaml
helm:
  valueFiles:
    - $values/repositories/values.yaml
    - $values/repositories/values-{{env}}.yaml
```

### 5.2 ApplicationSet 사용

대량의 애플리케이션을 생성할 때 ApplicationSet 사용:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: apps
spec:
  generators:
    - clusters: {}
  template:
    metadata:
      name: '{{name}}-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/repo
        targetRevision: HEAD
        path: apps/{{name}}
      destination:
        server: '{{server}}'
        namespace: default
```

## 트러블슈팅

### 애플리케이션이 동기화되지 않는 경우

1. 리포지토리 접근 권한 확인
2. 헬름 차트 경로 및 버전 확인
3. ArgoCD 로그 확인: `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller`

### Secret이 적용되지 않는 경우

1. Secret이 올바른 네임스페이스에 생성되었는지 확인
2. ArgoCD가 Secret을 읽을 권한이 있는지 확인
3. `argocd.argoproj.io/secret-type` 라벨이 올바르게 설정되었는지 확인
