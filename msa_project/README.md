#고양이와 강아지 중 하나를 투표하여 투표율을 보여주는 프로그램을 msa로 구성해보았다.
<img width="960" alt="투표화면" src="https://user-images.githubusercontent.com/88422822/170530399-56764b28-5f86-434a-95d9-ce6a5c649cc9.png">
<img width="947" alt="투표결과 화면" src="https://user-images.githubusercontent.com/88422822/170530407-0841fc2f-ad09-44a0-88d9-255e8ed95eae.png">


#1. redis에 투표한 내용을 전달받아 저장하는 역할을 하는 deployment를 생성한다.
```
#redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis:alpine
        name: redis
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
      - name: redis-data
        emptyDir: {}
```
#2. redis-deployment에 clusterIP로 서비스를 붙여서 클러스터 내부에 노출시킨다.

```
#redis-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: redis
  namespace: vote
spec:
  type: ClusterIP
  ports:
  - name: "redis-service"
    port: 6379
    targetPort: 6379
  selector:
    app: redis
```
#3. 투표화면과 투표결과를 전달하는 프로그램을 deployment로 생성한다.
```
#vote-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: result
  name: result
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - image: dockersamples/examplevotingapp_result:before
        name: result
        ports:
        - containerPort: 80
          name: result
  ```
#4. vote-deployment를 클러스터 외부에 nodePort서비스로 노출시킨다.
```
#vote-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: vote
  name: vote
  namespace: vote
spec:
  type: NodePort
  ports:
  - name: "vote-service"
    port: 5000
    targetPort: 80
    nodePort: 31000
  selector:
    app: vote
```
#5. db에 투표 결과를 저장하는 deployment를 생성한다.
```
#db-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: db
  name: db
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - image: postgres:9.4
        name: postgres
        env:
        - name: POSTGRES_USER
          value: postgres
        - name: POSTGRES_PASSWORD
          value: postgres
        - name: POSTGRES_HOST_AUTH_METHOD
          value: trust
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-data
      volumes:
      - name: db-data
        emptyDir: {}
```
#6. db-deployment를 clusterIP서비스로 클러스터 내부에 노출시킨다.
```
#db-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: db
  name: db
  namespace: vote
spec:
  type: ClusterIP
  ports:
  - name: "db-service"
    port: 5432
    targetPort: 5432
  selector:
    app: db
```
#7. 투표결과를 보여주는 화면을 보여주는 기능을 deployment로 생성한다.
```
#result-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: result
  name: result
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - image: dockersamples/examplevotingapp_result:before
        name: result
        ports:
        - containerPort: 80
          name: result
```
#7. nodePort서비스로 result-deployment를 클러스터 외부에 노출시킨다.
```
#result-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: result
  name: result
  namespace: vote
spec:
  type: NodePort
  ports:
  - name: "result-service"
    port: 5001
    targetPort: 80
    nodePort: 31001
  selector:
    app: result
````
#8. redis-deployment에 저장된 투표결과를 db-deployment로 전달하는 기능을 deployment로 생성한다.
```
#worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: worker
  name: worker
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - image: dockersamples/examplevotingapp_worker
        name: worker
```
