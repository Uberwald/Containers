# ЛР 3. Kubernetes
## pg_configmap.yml
```go
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-configmap
  labels:
    app: postgres
data:
  POSTGRES_DB: "postgres"
```

## pg_service.yml
```go
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  labels:
    app: postgres
spec:
  type: NodePort
  ports:
   - port: 5432
  selector:
   app: postgres
```

## pg_deployment.yml
```go
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres-container
          image: postgres:14
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 100m
              memory: 128Mi
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-configmap
            - secretRef:
                name: postgres-secret
```
## postgres-secret.yml
```go
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  labels:
    app: postgres
type: Opaque
stringData:
  POSTGRES_USER: "postgres"
  POSTGRES_PASSWORD: "any_password_u_want"
```
## nextcloud-env.yml
```go
apiVersion: v1
kind: ConfigMap
metadata:
  name: nextcloud-env-configmap
  labels:
    app: nextcloud
data:
  NEXTCLOUD_UPDATE: "1"
  ALLOW_EMPTY_PASSWORD: "yes"
```
## nextcloud.yml
```go
apiVersion: v1
kind: Secret
metadata:
  name: nextcloud-secret
  labels:
    app: nextcloud
type: Opaque
stringData:
  NEXTCLOUD_ADMIN_PASSWORD: "literally_any_password"
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nextcloud
  labels:
    app: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
  template:
    metadata:
      labels:
        app: nextcloud
    spec:
      containers:
      - name: nextcloud
        image: docker.io/nextcloud:stable-apache
        resources:
          limits:
            cpu: 500m
            memory: 256Mi
          requests:
            cpu: 250m
            memory: 128Mi
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        envFrom:
        - configMapRef:
            name: nextcloud-env-configmap
        env:       
        - name: POSTGRES_HOST
          value: postgres-service
        - name: POSTGRES_DB
          valueFrom:
            configMapKeyRef:
              name: postgres-configmap
              key: POSTGRES_DB
        - name: NEXTCLOUD_TRUSTED_DOMAINS
          value: "127.0.0.1"
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_PASSWORD
        - name: NEXTCLOUD_ADMIN_USER
          value: any_name_you_want
        - name: NEXTCLOUD_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nextcloud-secret
              key: NEXTCLOUD_ADMIN_PASSWORD
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 60
          periodSeconds: 60 
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 160
          periodSeconds: 60
        imagePullPolicy: IfNotPresent
      restartPolicy: Always
      dnsPolicy: ClusterFirst
```

<figure>
  <img
  src="https://github.com/Uberwald/GAN_study/blob/main/HW1/1.2.Picture/1.png"
  alt="Картинка 1">
  <figcaption>Картинка 1</figcaption>
</figure>
