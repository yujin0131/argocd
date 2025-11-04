# ArgoCD 프로젝트 구조 옵션

ArgoCD App of Apps 패턴을 구현하는 방법은 여러 가지가 있습니다. 여기서는 다양한 구조 옵션을 소개합니다.

## 옵션 1: 최상위 루트 애플리케이션 (권장)

루트 애플리케이션을 최상위에 두고, applications 디렉토리를 직접 참조합니다.

```
argocd/
├── root-app.yaml              # 최상위 루트 애플리케이션
├── applications/
│   ├── repositories/
│   ├── projects/
│   ├── policies/
│   └── secrets/
├── values/
│   ├── repositories/
│   ├── projects/
│   ├── policies/
│   └── secrets/
└── README.md
```

**장점:**
- 구조가 단순하고 명확함
- 루트 애플리케이션을 쉽게 찾을 수 있음
- bootstrap 디렉토리 불필요

**사용법:**
```bash
kubectl apply -f root-app.yaml
```

## 옵션 2: Bootstrap 디렉토리 (현재 구조)

루트 애플리케이션을 bootstrap 디렉토리에 분리합니다.

```
argocd/
├── bootstrap/
│   └── root-app.yaml
├── applications/
├── values/
└── README.md
```

**장점:**
- 초기 설정 파일이 분리되어 있음
- 대규모 프로젝트에서 구조가 명확함

**단점:**
- 디렉토리가 하나 더 추가됨
- 간단한 프로젝트에는 과할 수 있음

## 옵션 3: ApplicationSet 사용

ApplicationSet을 사용하면 루트 애플리케이션이 필요 없습니다.

```
argocd/
├── applicationsets/
│   └── appset.yaml           # ApplicationSet 정의
├── applications/
├── values/
└── README.md
```

**장점:**
- 여러 클러스터/네임스페이스에 자동으로 배포 가능
- 루트 애플리케이션 관리 불필요
- 대량의 애플리케이션 생성에 적합

**단점:**
- ApplicationSet 이해 필요
- 설정이 다소 복잡할 수 있음

## 옵션 4: 단일 애플리케이션 디렉토리

모든 애플리케이션을 하나의 디렉토리에 평면적으로 관리합니다.

```
argocd/
├── apps.yaml                 # 모든 애플리케이션 정의
├── values/
│   ├── repositories.yaml
│   ├── projects.yaml
│   ├── policies.yaml
│   └── secrets.yaml
└── README.md
```

**장점:**
- 매우 단순한 구조
- 소규모 프로젝트에 적합

**단점:**
- 확장성이 떨어짐
- 애플리케이션이 많아지면 관리가 어려움

## 권장 구조

**소규모 프로젝트**: 옵션 1 (최상위 루트 애플리케이션)
**중규모 프로젝트**: 옵션 1 또는 옵션 2
**대규모/멀티 클러스터**: 옵션 3 (ApplicationSet)

## 구조 변경 방법

현재 구조에서 다른 구조로 변경하려면:

1. 원하는 구조 선택
2. 파일 위치 조정
3. root-app.yaml의 path 설정 업데이트
