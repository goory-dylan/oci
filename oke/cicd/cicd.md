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
spring-topic
```

* DevOps 프로젝트 생성
```
Developer Services > DevOps > Create Project >
Project Name : Spring

Change Topic
spring-topic

Enable Logging
```

* Sample Source Download
```
curl https://start.spring.io/starter.tgz -d baseDir=spring -d name=spring -d artifactId=rest-service -d javaVersion=1.8 -d dependencies=web,actuator | tar -xzvf -

```

* Code Repository 생성
```
Name : spring-code

git clone <YourClonewithHTTPS URL>

username : <tenancy-name>/<username>
password : ${AuthToken_Value}

wget https://github.com/TheKoguryo/MuShop-storefront/archive/refs/tags/v2022.03.tar.gz
tar -xvzf v2022.03.tar.gz --strip-components=1 -C mushop-storefront-code-repo/

git config --global user.email "you@example.com"
git config --global user.name "Your Name"

cd spring-code
git add .
git commit -m "init"
git push
```

