# Logging 배포
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













참고
https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengviewingworkernodelogs.htm
