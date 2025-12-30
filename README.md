# KYEOL Platform GitOps

KYEOL 클러스터 플랫폼 애드온 및 ArgoCD 구성 레포지토리입니다.

## 디렉토리 구조

```
kyeol-platform-gitops/
├── argocd/
│   ├── bootstrap/           # ArgoCD 설치 매니페스트
│   └── app-of-apps/         # Root App + Projects
├── clusters/
│   ├── dev/                 # DEV 클러스터 addons
│   │   ├── values/          # Helm values 파일
│   │   └── addons/          # Kustomize 래퍼
│   ├── mgmt/                # MGMT 클러스터
│   ├── stage/               # STAGE (템플릿)
│   └── prod/                # PROD (템플릿)
└── common/                  # 공통 매니페스트
```

## 설치 순서

### 1. MGMT 클러스터에 ArgoCD 설치

```bash
kubectl config use-context arn:aws:eks:ap-southeast-2:ACCOUNT:cluster/min-kyeol-mgmt-eks
kubectl apply -k argocd/bootstrap/
```

### 2. ArgoCD 초기 비밀번호 확인

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 3. DEV 클러스터에 Addons 설치

```bash
kubectl config use-context arn:aws:eks:ap-southeast-2:ACCOUNT:cluster/min-kyeol-dev-eks
kubectl apply -k clusters/dev/addons/aws-load-balancer-controller/
kubectl apply -k clusters/dev/addons/external-dns/
kubectl apply -k clusters/dev/addons/metrics-server/
```

## IRSA 설정 필수

Helm values 파일의 `serviceAccount.annotations.eks.amazonaws.com/role-arn`을
Terraform output의 실제 ARN으로 교체해야 합니다.
