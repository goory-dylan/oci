# nginx sample 배포

1. namespace 생성
```
k create namespace nginx
```

2. deployment 배포
```
kubectl apply -f https://k8s.io/examples/application/deployment.yaml -n nginx


deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.io/library/nginx:1.14.2
        ports:
        - containerPort: 80
```
3. service expose
```
k expose deployment nginx-deployment --port=80 --type=LoadBalancer -n nginx

or

k create -f nginx-svc.yaml -n nginx

nginx-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-deployment
spec:
  externalTrafficPolicy: Cluster
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  sessionAffinity: None
  type: LoadBalancer
```

4. nginx 접속 IP 확인
```
k get svc -n nginx
```
