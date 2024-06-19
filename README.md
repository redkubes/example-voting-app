# example-voting-app

This repo contains the source code to build the `vote`, `worker` and `results` images for deploying the Docker samples [example-voting-app] in Otomi using the provided Helm charts in the Otomi Catalog.

<img title="Voting App" alt="App diagram" src="Voting App.jpg">

# Get started

## Building the images

Use the Build feature in Otomi to build the images with `mode-Docker`. Set the `path` to `./vote/Dockerfile` (or `./worker/Dockerfile` or `./result/Dockerfile`).

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

## Register the services

Register the `vote` and `result` services in Otomi and configure them for external exposure. 

### Vote

- Register the `vote` service 
- Set exposure to `External`


### Result

- Register the `<result>` service 
- Set exposure to `External`

## Network Policies

If `Network policies` are enabled, then register all services and configure the network policies:

### Postgres Database
- Create a new Netpol and select the ingress rule type.
- Add the selector label name `otomi.io/app`.
- Add the selector label value `<postgres-workload-name>`.
- Select AllowOnly.
- Add the namespace `<team-name>`, the selector label name `otomi.io/app` and the selector label value `<worker>`.
- Add the namespace `<team-name>`, the selector label name `otomi.io/app` and the selector label value `<result>`.

### Redis
- Create a new Netpol and select the ingress rule type.
- Add the selector label name `otomi.io/app`.
- Add the selector label value `<redis-workload-name>`.
- Select AllowOnly.
- Add the namespace `<team-name>`, the selector label name `otomi.io/app` and the selector label value `<worker>`.
- Add the namespace `<team-name>`, the selector label name `otomi.io/app` and the selector label value `<vote>`.