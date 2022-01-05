# OKE

## OKE 클러스터 배포


### Bastion Instacne 생성
1. 생성시 Network 를 OKE VCN 과 동일한 VCN 으로 구성
2. SSH 접근을 위해 Public subnet 설정
3. Security List 의 Ingress Rule에 port 22 허용
4. Security List 의 Egress Rule 에 All Protocol 허용
5. SSH 접속
6. oci 설치
```
sudo yum install python36-oci-cli
```
7. kubectl 설치
```
sudo yum install kubectl -y
```
