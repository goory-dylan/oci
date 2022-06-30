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

* Sample Source Download
```
git clone https://github.com/goory-dylan/spring-sample.git

chmod 755 mvnw

cd spring-sample
rm .git
cp -R * ../spring-code/
cp -R .mvn ../spring-code/
```

* Code Repository 생성
```
Name : spring-code

git clone <YourClonewithHTTPS URL>

username : <tenancy-name>/oracleidentitycloudservice/<username>
password : ${AuthToken_Value}


git config --global user.email "you@example.com"
git config --global user.name "Your Name"

cd spring-code
git add .
git commit -m "init"
git push
```

* Build Pipeline 생성
```
Name : spring-build-pipeline

Manage Build추가

Stage name : Build-Stage
Primary Code Repository 선택
```


* Build spec 작성 (build_spec.yaml)
```
version: 0.1
component: build
timeoutInSeconds: 6000
shell: bash
env:
  variables:
    appName: "spring-app"
  exportedVariables:
    - APP_NAME
    - OCIR_PATH
    - TAG

steps:
  - type: Command
    name: "Init exportedVariables"
    timeoutInSeconds: 4000
    command: |
      APP_NAME="spring-app"
      #APP_NAME=`grep '"name"' package.json | cut -d '"' -f 4 | head -n 1`
      #
  - type: Command
    name: "Build Source"
    timeoutInSeconds: 4000
    command: |
      echo none

  - type: Command
    name: "Define Image Tag - Commit ID"
    timeoutInSeconds: 30
    command: |
      COMMIT_ID=`echo ${OCI_TRIGGER_COMMIT_HASH} | cut -c 1-7`
      BUILDRUN_HASH=`echo ${OCI_BUILD_RUN_ID} | rev | cut -c 1-7`
      [ -z "$COMMIT_ID" ] && TAG=$BUILDRUN_HASH || TAG=$COMMIT_ID

  - type: Command
    name: "Define OCIR Path"
    timeoutInSeconds: 30
    command: |
      TENANCY_NAMESPACE=`oci os ns get --query data --raw-output`
      REPO_NAME=$appName
      OCIR_PATH=$OCI_RESOURCE_PRINCIPAL_REGION.ocir.io/$TENANCY_NAMESPACE/$REPO_NAME

  - type: Command
    timeoutInSeconds: 400
    name: "Containerize"
    command: |
      export DOCKER_BUILDKIT=1
      ./mvnw clean package
      docker build -t new-generated-image .
      docker images

  - type: Command
    name: "Check exportedVariables"
    timeoutInSeconds: 30
    command: |
      [ -z "$APP_NAME" ] && APP_NAME=unknown
      [ -z "$OCIR_PATH" ] && OCIR_PATH=unknown
      [ -z "$TAG" ] && TAG=unknown
      echo "APP_NAME: " $APP_NAME
      echo "OCIR_PATH: " $OCIR_PATH
      echo "TAG: " $TAG

outputArtifacts:
  - name: output-image
    type: DOCKER_IMAGE
    location: new-generated-image    
```
