# example-voting-app

This repo contains the source code to build the `vote`, `worker` and `results` images for deploying the Docker samples [example-voting-app] in Otomi using the provided Helm charts in the Otomi Catalog.

<img title="Voting App" alt="App diagram" src="Voting App.jpg">

# Get started

## Building the images

Use the [Build feature in Otomi](https://otomi.io/docs/get-started/labs/lab-6#build-the-blue-image) to build the images with `Docker` mode for the `result`,`vote` and `worker` components: 
1. Set the build name (ex. `voting-app-vote`)
2. Set the `Repo URL` to `https://github.com/redkubes/example-voting-app.git`
3. Set the `path` to `./vote/Dockerfile` 
4. Repeat step 1,2 and 3 for the other 2 components
    - use `./result/Dockerfile` and `./worker/Dockerfile` paths respectively


## Create a Redis cluster and a PostgreSQL database

Use the `postgresql` and the `redis` charts in the Otomi `Catalog` to create a Redis master-replica cluster and a PostgreSQL database. 
- **Postgresql**:
  - Name: `<postgesql app name>` (E.g.`voting-app-psql`).
  - Click `Submit` (can use default values for this example)
- **Redis**:
  - Name: `<redis app name>` (E.g.`voting-app-redis`).
  - For this demo, Redis authentication needs to be turned off by setting 
      ```yaml
    auth:
      enabled: false
    ```
    in the chart `Values` editor.
  - Click `Submit` and then `Deploy`
## Deploy the Vote app

Use the `k8s-deployment` chart to deploy the vote app. Use the following values:

- Name: `voting-app-vote`
- Update Values:
  ```yaml
  containerPorts:
    - name: http
      containerPort: 80
      protocol: TCP
  env:
    - name: REDIS_HOST
      value: <redis-cluster-name>-master # E.g. voting-app-redis-master
  ```
- Click `Submit`

## Deploy the Worker app

Use the `k8s-deployment` chart to deploy the worker app. Use the following values:

- Name: `voting-app-worker`
- Update Values:
  ```yaml
  containerPorts:
    - name: http
      containerPort: 80
      protocol: TCP
  env:
    - name: DATABASE_USER
      valueFrom:
        secretKeyRef:
          name: <psql-cluster-name>-superuser # E.g. voting-app-psql-superuser
          key: username
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: <psql-cluster-name>-superuser # E.g. voting-app-psql-superuser
          key: password
    - name: REDIS_HOST
      value: <redis-cluster-name>-master      # E.g. voting-app-redis-master
    - name: DATABASE_HOST
      value: <psql-cluster-name>-rw           # E.g. voting-app-psql-rw
  ```
- Click `Submit`

## Deploy the Result app

Use the `k8s-deployment` chart to deploy the result app. Use the following values:

- Name: `voting-app-result`
- Update Values:
  ```yaml
  containerPorts:
    - name: http
      containerPort: 80
      protocol: TCP
  env:
    - name: DATABASE_USER
      valueFrom:
        secretKeyRef:
          name: <psql-cluster-name>-superuser # E.g. voting-app-psql-superuser
          key: username
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: <psql-cluster-name>-superuser # E.g. voting-app-psql-superuser
          key: password
    - name: DATABASE_HOST
      value: <psql-cluster-name>-rw           # E.g. voting-app-psql-rw
  ```
- Click `Submit` and then `Deploy`

## Register the services

Register the `vote` and `result` services in Otomi and configure them for external exposure. 

### Vote

- Register the `vote` service 
- Set exposure to `External`
- Click `Submit`

### Result

- Register the `result` service 
- Set exposure to `External`
- Click `Submit` and then `Deploy`

## Network Policies

If `Network policies` are enabled, then register all services and configure the network policies:

### Postgres Database
- Create a new Netpol and select the ingress rule type.
- Add the selector label name `otomi.io/app`.
- Add the selector label value `<postgres-workload-name>` (E.g. `voting-app-psql`).
- Select AllowOnly.
- Add the namespace `<team-name>` (E.g. `team-demo`), the selector label name `otomi.io/app` and the selector label value `<worker>` (E.g. `voting-app-worker`).
- Add the namespace `<team-name>` (E.g. `team-demo`), the selector label name `otomi.io/app` and the selector label value `<result>` (E.g. `voting-app-result`).
- Click `Submit`

### Redis
- Create a new Netpol and select the ingress rule type.
- Add the selector label name `otomi.io/app`.
- Add the selector label value `<redis-workload-name>` (E.g. `voting-app-redis`).
- Select AllowOnly.
- Add the namespace `<team-name>` (E.g. `team-demo`), the selector label name `otomi.io/app` and the selector label value `<worker>` (E.g. `voting-app-worker`).
- Add the namespace `<team-name>` (E.g. `team-demo`), the selector label name `otomi.io/app` and the selector label value `<vote>` (E.g. `voting-app-vote`).
- Click `Submit` and then `Deploy`