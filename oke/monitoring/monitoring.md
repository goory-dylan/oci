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
5. 배포 상태 확인
```
k get all -n monitoring
```
6. Garafana Loadbalancer 생성
```
k get svc -n monitoring
k edit svc kube-prometheus-stack-{RANDOM_NUMBER}-grafana

# 타입변경 후 OCI의 Loadbalancer가 생성되면 PublicIP 를 확인하여 접속
type: LoadBalancer
```
7. Garafana Password 확인
```
k get secret -n monitoring | grep grafana
k get secret kube-prometheus-stack-{RANDOM_NUMBER}-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode; echo

ID: admin
PWD: prom-operator
```
8. Grafana Dashboard 배포
```
Grafana Dashboard 접속
https://grafana.com/grafana/dashboards/
필요한 Dashboard 검색하여 Code를 확인하고 Garafana 콘솔에서 "+" 클릭후 Dashboard Import 항목에 Code 입력
```
