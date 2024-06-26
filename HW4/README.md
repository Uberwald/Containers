# ЛР 4. More Kubernetes

# Проверка соответствия Kubernetes конфигураций условиям

### Условие 1: Минимум два `Deployment`, по количеству сервисов
В проекте есть два `Deployment`:
- `pg_deployment.yml` для PostgreSQL.
- `nextcloud.yml` для Nextcloud.

### Условие 2: Кастомный образ для минимум одного `Deployment`
Кастомный образ используется для `nextcloud`:
- В `nextcloud.yml` указан образ `my-nextcloud-image`, который собирается из Dockerfile.

### Условие 3: Минимум один `Deployment` должен содержать в себе контейнер и инит-контейнер
`pg_deployment.yml` содержит и основной контейнер, и init-контейнер:
```yaml
initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'echo "Initializing database..."; sleep 10;']
```

### Условие 4: Минимум один Deployment должен содержать volume
```yaml
volumes:
  - name: postgres-storage
    emptyDir: {}
```

### Условие 5: Обязательно использование ConfigMap и/или Secret
Оба Deployment используют и ConfigMap, и Secret:   


pg_deployment.yml использует postgres-configmap и postgres-secret.   

nextcloud.yml использует nextcloud-env-configmap и nextcloud-secret.

### Условие 6: Обязательно Service хотя бы для одного из сервисов
Существует Service для PostgreSQL:   

pg_service.yml.

### Условие 7: Liveness и/или Readiness пробы минимум в одном из Deployment

В nextcloud.yml указаны livenessProbe и readinessProbe:
```yaml
livenessProbe:
  exec:
    command:
      - curl
      - http://localhost/status.php
  initialDelaySeconds: 60
  periodSeconds: 30
readinessProbe:
  exec:
    command:
      - curl
      - http://localhost/status.php
  initialDelaySeconds: 60
  periodSeconds: 30

```

### Условие 8: Обязательно использование лейблов
Лейблы используются во всех ваших манифестах:   

Например, в pg_configmap.yml:

```yaml
labels:
  app: postgres

```




## Docker
```go
FROM docker.io/nextcloud:stable-apache

RUN apt-get update \
    && apt-get install -y \
        curl \
        vim \
    && rm -rf /var/lib/apt/lists/*

# COPY <локальный_файл_или_директория> <путь_в_контейнере>

# RUN sed -i 's/Listen 80/Listen 8080/' /etc/apache2/ports.conf

CMD ["apache2-foreground"]
```

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

## pg_secret.yml
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
## pg_deployment.yml
```go
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  labels:
    app: postgres
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
      initContainers:
        - name: init-db
          image: busybox
          command: ['sh', '-c', 'echo "Initializing database..."; sleep 10;']
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
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-configmap
            - secretRef:
                name: postgres-secret
      volumes:
        - name: postgres-storage
          emptyDir: {}
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
  POSTGRES_HOST: "postgres-service"
  POSTGRES_DB: "postgres"
  NEXTCLOUD_TRUSTED_DOMAINS: "127.0.0.1"
```

## nextcloud-secret.yml
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
  POSTGRES_USER: "postgres"
  POSTGRES_PASSWORD: "any_password_u_want"
```

## nextcloud-deployment.yml
```go
apiVersion: apps/v1
kind: Deployment
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
          image: my-nextcloud-image
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
            - secretRef:
                name: nextcloud-secret
          livenessProbe:
            exec:
              command:
                - curl
                - http://localhost/status.php
            initialDelaySeconds: 60
            periodSeconds: 30
          readinessProbe:
            exec:
              command:
                - curl
                - http://localhost/status.php
            initialDelaySeconds: 60
            periodSeconds: 30
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      dnsPolicy: ClusterFirst
```

<figure>
  <img
  src="https://github.com/Uberwald/Containers/blob/main/HW4/1.jpg"
  alt="Картинка 1">
  <figcaption>Картинка 1</figcaption>
</figure>

<figure>
  <img
  src="https://github.com/Uberwald/Containers/blob/main/HW4/2.jpg"
  alt="Картинка 1">
  <figcaption>Картинка 1</figcaption>
</figure>

<figure>
  <img
  src="https://github.com/Uberwald/Containers/blob/main/HW4/3.jpg"
  alt="Картинка 1">
  <figcaption>Картинка 1</figcaption>
</figure>

<figure>
  <img
  src="https://github.com/Uberwald/Containers/blob/main/HW4/4.jpg"
  alt="Картинка 1">
  <figcaption>Картинка 1</figcaption>
</figure>

<figure>
  <img
  src="https://github.com/Uberwald/Containers/blob/main/HW4/5.jpg"
  alt="Картинка 1">
  <figcaption>Картинка 1</figcaption>
</figure>
