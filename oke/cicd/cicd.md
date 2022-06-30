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


* Build spec 작성 (build_spec.yaml / 작성 후 git push)
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

* Start manual build 실행
```
Build Pipelines 에서 manual run 을 이용하여 이상 유무 확인
```

* OCIR 이미지 저장을 위한 stage 추가
```
build pipeline > Add stage > Delivery Artifact Stage
Stage name : deliver-generated-image

Create Artifact
Name: generated_image_with_tag
Image Path: ${OCIR_PATH}:${TAG}

Name: generated_image_with_latest
Image Path: ${OCIR_PATH}:latest


Build config/result Artifact name : output-image
```

* Deploy Artifact 생성
```
DevOps 프로젝트에서 Artifacts 선택
Add artifact > Kubernetes manifest
Name : k8s_spring_deploy_template
Type : Kubernetes manifest
Artifact source : Inline

Value : 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: spring-boot-greeting
  name: spring-boot-greeting-deployment
  namespace: spring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-boot-greeting
  template:
    metadata:
      labels:
        app: spring-boot-greeting
    spec:
      containers:
      - name: spring-boot-greeting
        image: ${OCIR_PATH}:${TAG}
      imagePullSecrets:
      - name: ocir-secret
      restartPolicy: Always
```

* Kubernetes Environment 등록
```
Devops 프로젝트 > Environments > Create Environments

Name : cluster1
Region, Compartment, Cluster 선택
```


* Deploy Pipeline 생성
```
Devops 프로젝트에서 Deployment Pipelines 선택
Name : spring-deployment-pipeline

Apply manifest to your Kubernetes Cluster
Add Stage


```

* namespace, secret 생성
```
spring namespace 를 미리 생성하고 ocir-secret 생성
kubectl create ns spring

kubectl create secret docker-registry ocir-secret \
  --docker-username="${OCIR_USER_NAME}" \
  --docker-password="${AUTH_TOKEN}" \
  --docker-email="${E-MAIL}" \
  --docker-server=ap-seoul-1.ocir.io
```
