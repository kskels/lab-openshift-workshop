This hands-on lab helps users to get familiar with [deployment strategies](https://docs.openshift.com/container-platform/4.8/applications/deployments/deployment-strategies.html).

### Create a Project

```bash
export OPENSHIFT_USERNAME=`oc whoami`
oc new-project ${OPENSHIFT_USERNAME}-proj5
```

### Rolling Strategy

#### Create an Application

Create example application with multiple replicas using *RollingUpdate* as deployment strategy. We will use conservative parameters for `maxSurge` and `maxUnavailable` for update to take slightly longer to be able to observe the process.

Notice `v1` image version used `quay.io/kskels/deployment-example:v1` for the application.

Create example deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 10
  selector:
    matchLabels:
      app: example-app

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: "10%" 
      maxUnavailable: "5%" 

  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-app
        image: quay.io/kskels/deployment-example:v1
        ports:
        - containerPort: 8080
```

Create example service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-app
spec:
  ports:
    - name: 8080-tcp
      port: 8080
      targetPort: 8080
  selector:
    app: example-app
```

Create example route

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: example-app
spec:
  port:
    targetPort: 8080-tcp
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: example-app
    weight: 100
```

Verify that application is successfully deployed.

```bash
oc rollout status deploy/example-app

oc get pods
oc get route
```

Access the application via route and make sure `v1` version is running.

Check deployment rollout history of the application

```bash
oc rollout history deploy/example-app
```



#### Update the Deployment

In the deployment manifest, update the image version from `v1` to `v2`, the image should look as `quay.io/kskels/deployment-example:v2`.

Save and apply the updated manifest.

```bash
oc apply -f example-app.yaml
```

Observe upgrade process by checking the *Pods*

```bash
oc get pods
```

Check the rollout status

```bash
oc rollout status deploy/example-app
```

Access the application via route and verify that `v2` version is running.


#### Rollback the Deployment

Check the rollout history

```bash
oc rollout history deploy/example-app
```

Rollback the desired version

```bash
kubectl rollout history deploy/example-app --revision=1
```

Access the application via route and verify that `v1` version is running.


### Recreate Stategy

Delete previous deployment

```bash
oc delete -f example-app.yaml
```

Update the deployment manifest to use *Recreate* strategy and reset image version to `v1`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 10
  selector:
    matchLabels:
      app: example-app

  strategy:
    type: Recreate

  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-app
        image: quay.io/kskels/deployment-example:v1
        ports:
        - containerPort: 8080
```

Create the deployment

```bash
oc create -f example-app.yaml
```

Check for successful deployment

```bash
oc rollout status deploy/example-app
```

In the deployment manifest, update the image version from `v1` to `v2`, the image should look as `quay.io/kskels/deployment-example:v2`.

Save and apply the updated manifest.

```bash
oc apply -f example-app.yaml
```

Observe the *Pods* and note differences from *RollingUpdate*.

```bash
oc get pods
```
