# AKS 구성 실습
Azure Cli 적용되었다는 가정하에 진행( Azure Cli 설치 : https://docs.microsoft.com/ko-kr/cli/azure/install-azure-cli )

## 1. Azure 리소스그룹 생성
```Bash
az group create --name test-dev-rg --location koreacentral
```
  
## 2. AKS 클러스터 생성
```Bash
az aks create --resource-group test-dev-rg --name test-dev-aks --node-count 1 --enable-addons monitoring --generate-ssh-key
```
  
## 3. update kubeconfig
```Bash
az aks get-credentials --resource-group test-dev-rg --name test-dev-aks
```
  
## 4. AKS 생성 확인 / App 배포 테스트
```Bash
## node 확인
kubectl get node

## 테스트용 App 배포 ( yaml 파일 내용은 아래 azure-vote.yaml 내용 참고 )
kubectl apply -f azure-vote.yaml

## azure-vote-front Service 의 External-IP 확인하여 접속 테스트
kubectl get service azure-vote-front --watch

## 테스트용 App 삭제
kubectl delete -f azure-vote.yaml
```
예제) 테스트 App 배포 파일 - `azure-vote.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-back
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-back
        image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
        env:
        - name: ALLOW_EMPTY_PASSWORD
          value: "yes"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-front
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-front
        image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

## 5. AKS 삭제(Azure 리소스그룹 삭제)
```Bash
az group delete --name test-dev-rg --yes --no-wait
```
  
  
  
\# 참고사이트  
https://docs.microsoft.com/ko-kr/azure/aks/kubernetes-walkthrough
