This section helps users to get familiar with *Kubernetes* [deployment strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy).

You can additionally explore *OpenShift DeploymentConfig* object that provides similar but different method for fine-grained management over common user applications. See more at [Understanding Deployment and DeploymentConfig objects](https://docs.openshift.com/container-platform/4.8/applications/deployments/what-deployments-are.html).




### Create a Project

```bash
export OPENSHIFT_USERNAME=`oc whoami`
oc new-project ${OPENSHIFT_USERNAME}-proj5
```

### Rolling Strategy

Create an example application with multiple replicas using *RollingUpdate* as the deployment strategy.

We will use conservative parameters for `maxSurge` and `maxUnavailable` for update to take slightly longer to be able to observe the process. Notice `v1` image version used `quay.io/kskels/deployment-example:v1` for the application.

Copy the following contents into a `.yaml` file, for example `example-app-deployment.yaml`

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

Apply the manifest

```bash
oc apply -f example-app-deployment.yaml
```


Copy the following contents into a `.yaml` file, for example `example-app-service.yaml`

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

Apply the manifest

```bash
oc apply -f example-app-service.yaml
```


Copy the following contents into a `.yaml` file, for example `example-app-route.yaml`

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

Apply the manifest

```bash
oc apply -f example-app-route.yaml
```


Verify that application is successfully deployed

```bash
oc rollout status deploy/example-app
oc get pods
```

Access the application via route and make sure `v1` version is running.

```bash
oc get route
```

Check the deployment rollout history of the application

```bash
oc rollout history deploy/example-app
```


#### Update the Deployment

In the deployment manifest, update the image version from `v1` to `v2`, the image should look as `quay.io/kskels/deployment-example:v2`. Save and apply the updated manifest

```bash
oc apply -f example-app-deployment.yaml
```

Observe upgrade process by checking the *Pods*

```bash
oc get pods
```

Check the rollout status

```bash
oc rollout status deploy/example-app
```

Access the application via route and verify that `v2` version is running

```bash
oc get route
```

#### Rollback the Deployment

Check the rollout history

```bash
oc rollout history deploy/example-app
```

Rollback the desired version

```bash
oc rollout undo deploy/example-app
```

Access the application via route and verify that `v1` version is running

```bash
oc get route
```


### Recreate Stategy

Delete previous deployment

```bash
oc delete -f example-app-deployment.yaml
```

Update the deployment manifest to use *Recreate* strategy and reset image version to `v1`

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

Apply the manifest

```bash
oc apply -f example-app-deployment.yaml
```

Check for successful deployment

```bash
oc rollout status deploy/example-app
```

In the deployment manifest, update the image version from `v1` to `v2`, the image should look as `quay.io/kskels/deployment-example:v2`.

Save and apply the updated manifest

```bash
oc apply -f example-app-deployment.yaml
```

Observe the *Pods* and note differences from *RollingUpdate*.

```bash
oc get pods
```
