This hands-on lab helps users to get familiar with *Services* and *Routes*.


If any issues encountered to copy the manifest contents into the terminal, you may optionally clone the repo and you will find all the snippets under `deploy/labs`

```bash
git clone https://github.com/kskels/aspnet-core-app
```

### Create a Project

```bash
export OPENSHIFT_USERNAME=`oc whoami`
oc new-project ${OPENSHIFT_USERNAME}-proj4
```

### Services

Services provide internal load-balancing and service discovery across pods. See [Service](https://kubernetes.io/docs/concepts/services-networking/service/) for more information.

We will use a sample `.NET` application with *Redis* as a backend to explore services.

#### Deploy Redis

Create and save `redis-deployment.yaml` file with the following contents

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aspnet-core-redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aspnet-core-redis
  template:
    metadata:
      labels:
        app: aspnet-core-redis
    spec:
      containers:
        - image: registry.redhat.io/rhel8/redis-6
          name: aspnet-core-redis
          ports:
            - containerPort: 6379
```

Use the `oc` command line interface to create the object

```bash
oc apply -f redis-deployment.yaml
```

To expose the service, create and save `redis-service.yaml` with the following contents

```yaml
apiVersion: v1
kind: Service
metadata:
  name: aspnet-core-redis
spec:
  ports:
    - name: 6379-tcp
      port: 6379
      targetPort: 6379
      protocol: TCP
  selector:
    app: aspnet-core-redis
```

Use the `oc` command line interface to create the object

```bash
oc apply -f redis-service.yaml
```

Explore objects created using `get` and `describe` commands

```bash
oc get all
oc describe service/aspnet-core-redis
```

### Routes

Routes make services accessible to clients outside the environment via real-world URLs. See [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) for more information.


#### Deploy .NET Application

Create and save `aspnet-core-app-deployment.yaml` file with the following contents

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aspnet-core-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: aspnet-core-app
  template:
    metadata:
      labels:
        app: aspnet-core-app
    spec:
      containers:
        - image: quay.io/kskels/aspnet-core-app:latest
          name: aspnet-core-app
          ports:
            - containerPort: 8080
```

Use the `oc` command line interface to create the object

```bash
oc apply -f aspnet-core-app-deployment.yaml
```

Create and save `aspnet-core-app-service.yaml` with the following contents

```yaml
apiVersion: v1
kind: Service
metadata:
  name: aspnet-core-app
spec:
  ports:
    - name: 8080-tcp
      port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: aspnet-core-app
```

Use the `oc` command line interface to create the object

```bash
oc apply -f aspnet-core-app-service.yaml
```

Finally, create and save `aspnet-core-app-route.yaml` with the following contents

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: aspnet-core-app
  name: aspnet-core-app
spec:
  port:
    targetPort: 8080-tcp
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: aspnet-core-app
    weight: 100
```

Use the `oc` command line interface to create the object

```bash
oc apply -f aspnet-core-app-route.yaml
```

Explore newly created objects including *Services* and *Routes*.

```bash
oc get all

oc describe service/aspnet-core-app
oc describe route/aspnet-core-app

oc get routes
```

You should now be able to access the application via web browser.

Explore services from one of the `aspnet-core-app` *Pods*

```bash

oc get pods
NAME                                READY   STATUS    RESTARTS   AGE
aspnet-core-app-6787fb8bcd-95d4d    1/1     Running   0          6m55s
aspnet-core-app-6787fb8bcd-ps6b7    1/1     Running   0          6m55s
aspnet-core-app-6787fb8bcd-s5ttw    1/1     Running   0          6m55s
aspnet-core-redis-84c6df894-4f5q4   1/1     Running   0          27m

oc exec -it aspnet-core-app-6787fb8bcd-95d4d -- bash
bash-4.4$ env |grep HOST
HOSTNAME=aspnet-core-app-6787fb8bcd-95d4d
ASPNET_CORE_APP_SERVICE_HOST=172.30.176.195
ASPNET_CORE_REDIS_SERVICE_HOST=172.30.4.101

bash-4.4$ env |grep PORT
ASPNET_CORE_APP_PORT_8080_TCP_PORT=8080
ASPNET_CORE_REDIS_PORT_6379_TCP_PROTO=tcp
(...)
```
### Further Reading

Explore [Ingress Controller](https://docs.openshift.com/container-platform/4.8/networking/ingress-operator.html), [route configuration](https://docs.openshift.com/container-platform/4.8/networking/routes/route-configuration.html) and [secured routes](https://docs.openshift.com/container-platform/4.8/networking/routes/secured-routes.html) to learn more.

### Review from Web Console

Switch back to the *Console* and explore newly created objects.
