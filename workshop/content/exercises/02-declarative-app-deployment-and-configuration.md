This hands-on lab helps users to get familiar with declarative application deplopyment and configuration.

We will be using YAML manfiests for application deployments. While JSON is also supported, YAML is more wide-spread and tends to be more user-friendly. Please take a look at the Kubernetes [configuration tips](https://kubernetes.io/docs/concepts/configuration/overview/) for more information.

### Create a New Project

```bash
oc new-project user2-project3
```

### Deploy the Spring Boot Application

The following is a single replica deployment of a container image running Spring Boot application. Copy the contents and save them as a `.yaml` file, for example `spring-app-deployment.yaml`.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: quay.io/kskels/spring-app:latest
        ports:
        - containerPort: 8080
``` 

Use the `oc` command line interface to deploy the manifest.

```bash
oc apply -f spring-app-deployment.yaml
```

Once applied, explore objects created by the `Deployment` configuration. Observe three objects created `pod`, `replicaset`, and `deployment`.

```bash
oc get all
```

You can further inspect each object with `describe` and `get` commands.

```bash
oc describe deploy/spring-app
oc get -o yaml deploy/spring-app
```

For futher exploring, once the `Pod` is in a `Running` state, you are able to access the pod itself.

```bash
oc get pods
NAME                        READY   STATUS    RESTARTS   AGE
spring-app-bbf96556-zlznz   1/1     Running   0          9m14s

oc exec -it spring-app-bbf96556-zlznz -- bash
[jboss@spring-app-bbf96556-zlznz ~]$ env
(...)
```

### Labels and Selectors

Please see general information on [labels and selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

From more practical view, switch between the namespaces `user2-project1` and `user2-project3` and observe the labels automatically created by OpenShift when applications are deployed via `oc new-app` and the Web UI. 

Here are some [common labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/) for Kuberentes.

These labels can be setup as part of the deployment, for example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app.openshift.io/runtime: java
    app.openshift.io/runtime-version: openjdk-11-ubi8
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: quay.io/kskels/spring-app:latest
        ports:
        - containerPort: 8080
```

Use the `oc` command line interface to apply the latest manifest.

```bash
oc apply -f spring-app-deployment.yaml
```


It is now possible to filter deployments based on runtime `java`

```bash
oc get deploy -l app.openshift.io/runtime=java
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
spring-app   1/1     1            1           76m
```

### Environment Variables

Use the `env` section under `spec.template.spec.containers.spring-app` to setup environment variables for a speicifc `Pod`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app.openshift.io/runtime: java
    app.openshift.io/runtime-version: openjdk-11-ubi8
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: quay.io/kskels/spring-app:latest
        ports:
        - containerPort: 8080

        env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_VERSION
          value: "v1.2"
```


Use the `oc` command line interface to apply the latest manifest.

```bash
oc apply -f spring-app-deployment.yaml
```

```bash
oc get pods
NAME                         READY   STATUS    RESTARTS   AGE
spring-app-787b9dbbc-xtnxc   1/1     Running   0          96s

oc exec -it spring-app-787b9dbbc-xtnxc -- bash
[jboss@spring-app-787b9dbbc-xtnxc ~]$ env |grep DEMO
DEMO_GREETING=Hello from the environment
DEMO_VERSION=v1.2
```


### Secrets

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. See [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) for more information.

As examples, for CI/CD workloads you would be storing credentials to Docker repositories to push/pull images, storing tokens to artifact systems to upload logs and build artifacts, and similarly in production systems to keep access credentials towards databases, etc.

For the pod/application, the `Secrets` can be accessed via container environment variables and as files in a volume mounted to a container.

#### Create a Secret

Copy the contents of the sample and save as a `.yaml` file, for example `spring-app-secret.yaml`.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: spring-app-secret
type: kubernetes.io/basic-auth
stringData:
  username: tekton
  password: tekton123!
  url: https://artifactory.apps.ocp4.kskels.com
```

Use the `oc` command line interface to create the secret

```bash
oc create -f spring-app-secret.yaml
```

Observe newly crated secret among system pre-created ones for the project.

```bash
oc get secrets
```


#### Add Secrets to the Deployment

Update the deployment with both environment variables and volume mounts using the secret.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app.openshift.io/runtime: java
    app.openshift.io/runtime-version: openjdk-11-ubi8
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: quay.io/kskels/spring-app:latest
        ports:
        - containerPort: 8080

        env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_VERSION
          value: "v1.2"

        # envs using spring-app-secret
        - name: ARTF_USERNAME
          valueFrom:
            secretKeyRef:
              name: spring-app-secret
              key: username
        - name: ARTF_PASSWORD
          valueFrom:
            secretKeyRef:
              name: spring-app-secret
              key: password

        # mount volumes to specfic contaiers
        volumeMounts:
        - name: spring-app-secret
          mountPath: "/etc/secrets/spring-app"
          readOnly: true

      # define volumes
      volumes:
      - name: spring-app-secret
        secret:
          secretName: spring-app-secret
```

Use the `oc` command line interface to apply the latest manifest.

```bash
oc apply -f spring-app-deployment.yaml
```

Explore the changes from the inside of the container.

```bash
oc get pods
NAME                          READY   STATUS    RESTARTS   AGE
spring-app-65d6b6d9dc-9s54z   1/1     Running   0          63s

oc exec -it spring-app-65d6b6d9dc-9s54z -- bash
[jboss@spring-app-65d6b6d9dc-9s54z ~]$ env |grep ARTF
ARTF_PASSWORD=tekton123!
ARTF_USERNAME=tekton

[jboss@spring-app-65d6b6d9dc-9s54z ~]$ cd /etc/secrets/spring-app/
[jboss@spring-app-65d6b6d9dc-9s54z spring-app]$ ls
password  url  username

[jboss@spring-app-65d6b6d9dc-9s54z spring-app]$ cat password
tekton123!
```

### Health Probes

Health probes is a mechanism to detect and handle unhealthy containers. See more information at [monitoring application health](https://docs.openshift.com/container-platform/4.8/applications/application-health.html).

In the following example we will setup a `readinessProbe` to ensure the application's web interface is available.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app.openshift.io/runtime: java
    app.openshift.io/runtime-version: openjdk-11-ubi8
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: quay.io/kskels/spring-app:latest
        ports:
        - containerPort: 8080

        env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_VERSION
          value: "v1.2"

        # envs using spring-app-secret
        - name: ARTF_USERNAME
          valueFrom:
            secretKeyRef:
              name: spring-app-secret
              key: username
        - name: ARTF_PASSWORD
          valueFrom:
            secretKeyRef:
              name: spring-app-secret
              key: password

        # mount volumes to specfic contaiers
        volumeMounts:
        - name: spring-app-secret
          mountPath: "/etc/secrets/spring-app"
          readOnly: true

        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 8080
            scheme: HTTP

      # define volumes
      volumes:
      - name: spring-app-secret
        secret:
          secretName: spring-app-secret
```

Use the `oc` command line interface to apply the latest manifest.

```bash
oc apply -f spring-app-deployment.yaml
```

If you observe the `Pod` shortly after applying the manifest you will notice that it will take slightly longer for the application to reach `1/1` readiness state; the system will poll the HTTP enpoint until successful reply is received.

```bash
oc get pods
NAME                          READY   STATUS    RESTARTS   AGE
spring-app-5fddc47864-xx5p8   0/1     Running   0          3s
spring-app-65d6b6d9dc-pk992   1/1     Running   0          28m
```


### Resource Limits

You can optionally specify how much CPU, memory, and local ephemeral storage (if your administrator enabled the ephemeral storage technology preview) each container needs in order to better schedule pods in the cluster and ensure satisfactory performance.

See more documentation about [quotas and limits](https://docs.openshift.com/online/pro/dev_guide/compute_resources.html).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app.openshift.io/runtime: java
    app.openshift.io/runtime-version: openjdk-11-ubi8
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: quay.io/kskels/spring-app:latest
        ports:
        - containerPort: 8080

        env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_VERSION
          value: "v1.2"

        # envs using spring-app-secret
        - name: ARTF_USERNAME
          valueFrom:
            secretKeyRef:
              name: spring-app-secret
              key: username
        - name: ARTF_PASSWORD
          valueFrom:
            secretKeyRef:
              name: spring-app-secret
              key: password

        # mount volumes to specfic contaiers
        volumeMounts:
        - name: spring-app-secret
          mountPath: "/etc/secrets/spring-app"
          readOnly: true

        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 8080
            scheme: HTTP

        resources:
          requests:
            memory: "2Gi"
            cpu: "500m"
          limits:
            memory: "4Gi"
            cpu: "1000m"

      # define volume
      volumes:
      - name: spring-app-secret
        secret:
          secretName: spring-app-secret
```

Use the `oc` command line interface to apply the latest manifest.

```bash
oc apply -f spring-app-deployment.yaml
```

Observe the resource constraints with

```bash
oc get pods
NAME                          READY   STATUS    RESTARTS   AGE
spring-app-66f58bcc98-4q526   1/1     Running   0          66s

oc describe po/spring-app-66f58bcc98-4q526
(...)
```