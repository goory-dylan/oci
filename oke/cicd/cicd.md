# Start CI/CD
OKE Cluster가 배포되어 있고 Bastion 서버가 생성되어 있는 상태에서 시작

* Dynamic Group 생성 (OKE가 배포되어 있는 Compartment에서 생성)
Name : CoderepoDynamicGroup
```
ALL {resource.type = 'devopsrepository', resource.compartment.id = '<YourCompartmentOCID>'}
```
Name : ConnectionDynamicGroup
```
ALL {resource.type = 'devopsconnection', resource.compartment.id = '<YourCompartmentOCID>'}
```
Name : BuildDynamicGroup
```
ALL {resource.type = 'devopsbuildpipeline', resource.compartment.id = '<YourCompartmentOCID>'}
```
Name : DeployDynamicGroup
```
ALL {resource.type = 'devopsdeploypipeline', resource.compartment.id = '<YourCompartmentOCID>'}
```

* Policy 설정 (OKE가 배포되어 있는 Compartment에서 생성)
 
Name : DevOps-compartment-policy
```
Allow dynamic-group CoderepoDynamicGroup to manage devops-family in compartment <YourCompartmentName>
Allow dynamic-group BuildDynamicGroup to manage repos in compartment <YourCompartmentName>
Allow dynamic-group BuildDynamicGroup to read secret-family in compartment <YourCompartmentName>
Allow dynamic-group BuildDynamicGroup to manage devops-family in compartment <YourCompartmentName>
Allow dynamic-group BuildDynamicGroup to manage generic-artifacts in compartment <YourCompartmentName>
Allow dynamic-group BuildDynamicGroup to use ons-topics in compartment <YourCompartmentName>
Allow dynamic-group DeployDynamicGroup to manage all-resources in compartment <YourCompartmentName>
Allow dynamic-group ConnectionDynamicGroup to read secret-family in compartment <YourCompartmentName>
```

* Policy 설정 (root Compartment에서 생성)

Name : DevOps-root-policy
```
Allow dynamic-group BuildDynamicGroup to manage repos in tenancy
```

* Notification Topic 생성
DevOps 프로젝트 생성시 필수 요구 사항이며 DevOps 파이프 라인 실행이 발생하는 주요 이벤트를 알려주기 위한 용도
```
Developer Services > Application Integration > Notifications
spring-devops-topic
```

* DevOps 프로젝트 생성
Developer Services > DevOps > Create Project >
Project Name : Spring

Change Topic
