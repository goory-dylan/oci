# Start CI/CD
```
OKE Cluster가 배포되어 있고 Bastion 서버가 생성되어 있는 상태에서 시작
```

1. Dynamic Group 생성
```
Name : OKE-DG (임의지정 가능)
Rule : instance.compartment.id = 'ocid1.tenancy.oc1..<unique-id>'
```
2. Policy 생성
```
name : OKE-POLICY (임의지정 가능)
POLICY BUILDER : allow dynamic-group <dynamic-group-name> to use log-content in tenancy
```
