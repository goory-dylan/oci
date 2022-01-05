# Monitoring Stack 배포

1. helm prometheus repository 추가
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
2. kubernetes 에 monitoring namespace 생성
```
k create namespace monitoring
```
3. helm 을 이용하여 monitoring namespace 에 prometheus stack 설치
```
helm install prometheus-community/kube-prometheus-stack --generate-name -n monitoring
```
4. helm 조회 및 삭제 (optional)
```
helm ls
helm uninstall {NAME}
```
