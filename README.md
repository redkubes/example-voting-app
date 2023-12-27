# example-voting-app

This repo contains the source code to build the `vote`, `worker` and `results` images for deploying the Docker samples [example-voting-app] in Otomi using the provided Helm charts in the Otomi Catalog.

# Get started

## Building the images

Use the Build feature in Otomi to build the images.

## Create a Redis cluster and a PostgreSQL database

Use the `postgresql` and the `redis` charts in the Otomi Catalog to create a Redis master-replica cluster and a PostgreSQL database. For this demo, Redis authentication needs to be turned off by setting `auth.enabled=false`.

## Deploy the Vote app

Use the `k8s-deployment` chart to deploy the vote app. Use the following values:

Name: `vote`

```yaml
containerPorts:
  - name: http
    containerPort: 80
    protocol: TCP
env:
  - name: REDIS_HOST
    value: <redis-cluster-name>-master
```

## Deploy the Worker app

Use the `k8s-deployment` chart to deploy the worker app. Use the following values:

Name: `worker`

```yaml
containerPorts:
  - name: http
    containerPort: 80
    protocol: TCP
env:
  - name: DATABASE_USER
    valueFrom:
      secretKeyRef:
        name: <psql-cluster-name>-superuser
        key: username
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: <psql-cluster-name>-superuser
        key: password
  - name: REDIS_HOST
    value: <redis-cluster-name>-master
  - name: DATABASE_HOST
    value: <psql-cluster-name>-rw
```

## Deploy the Result app

Use the `k8s-deployment` chart to deploy the result app. Use the following values:

Name: `result`

```yaml
containerPorts:
  - name: http
    containerPort: 80
    protocol: TCP
env:
  - name: DATABASE_USER
    valueFrom:
      secretKeyRef:
        name: <psql-cluster-name>-superuser
        key: username
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: <psql-cluster-name>-superuser
        key: password
  - name: DATABASE_HOST
    value: <psql-cluster-name>-rw
```

## Expose the services

Expose the `vote` and `result` services.