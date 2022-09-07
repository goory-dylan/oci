# OKE

## OKE 클러스터 배포


### Bastion Instacne 설정
1. 생성시 Network 를 OKE VCN 과 동일한 VCN 으로 구성
2. SSH 접근을 위해 Public subnet 설정
3. Security List 의 Ingress Rule에 port 22 허용
4. Security List 의 Egress Rule 에 All Protocol 허용
5. SSH 접속
6. oci 설치
```
sudo yum install python36-oci-cli -y
```
7. kubectl 설치
```
sudo yum install kubectl -y
```
8. helm 3 설치
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
9. .bashrc 설정
```
alias k=kubectl
export KUBECONFIG=$HOME/.kube/config

#현재 namespace 확인
alias wmi='kubectl config view --minify | grep namespace'

#namespace 변경
alias kcd='kubectl config set-context $(kubectl config current-context) --namespace'
```

## OKE 접속을 위한 oci config 설정
1. oci config 생성
```
oci setup config
Enter a location for your config [${HOME}/.oci/config]:
config 파일 위치 지정
입력: Enter (기본 위치 사용)
Enter a user OCID:
입력: OCI User의 ocid
ocid1.user.oc1..aaaaa.....qwmkasndkla
Enter a tenancy OCID
입력: OCI Tenancy의 oicd
ocid1.tenancy.oc1..aaaaa.....qwmkasndkla
Enter a region (e.g. eu-frankfurt-1, uk-london-1, us-ashburn-1, us-phoenix-1):
입력: 8 (8번은 Seoul region)
Do you want to generate a new RSA key pair?
RSA 보안 키 파일 생성
입력: Y
Enter a directory for your keys to be created [${HOME}/.oci]:
RSA 보안 키 파일 위치 지정
입력: Enter (기본 위치를 사용할 경우)
Enter a name for your key [oci_api_key]:
RSA Private key 파일 이름 지정
입력: Enter (기본 위치를 사용할 경우)
Enter a passphrase for your private key (empty for no passphrase):
입력: Enter (기본값 사용)

출처 : http://taewan.kim/tutorial_manual/handson_adw/05.preprocessing/4/
```
2. API Key 등록
```
OCI 콘솔에서 사용하고자 하는 유저 선택
API Keys 에서 Add API Key 클릭
앞서 생성한 RSA oci_api_key_public.pem 키 값 ADD
```
3. OKE Access Cluster 에서 Local Access 내용을 참고하여 kubeconfig 파일 생성
```
oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.ap-seoul-1.aaaaaaaa...wnzfnm --file $HOME/.kube/config --region ap-seoul-1 --token-version 2.0.0  --kube-endpoint PUBLIC_ENDPOINT

or

oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.ap-seoul-1.aaaaaaaa...wnzfnm --file $HOME/.kube/config --region ap-seoul-1 --token-version 2.0.0  --kube-endpoint PRIVATE_ENDPOINT
```
