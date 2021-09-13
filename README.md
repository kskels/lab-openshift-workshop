OpenShift Workshop
==================


This is content repository for an interactive workshop that helps users to get familiar with *Kubernetes* and *OpenShift Container Platform 4* core concepts focusing on declarative application deployment using `YAML` manifests.

See *YouTube* instructions video for information on [OpenShift Homeroom](https://www.youtube.com/watch?v=HxBfCRGCvyc&t=251s) learning environment and how to deploy this lab onto your own *OpenShift* cluster.

### Deploy the Workshop

Create a new project

```bash
$ oc new-project workshop
```

Init the scripts *Git* submodule

```bash
$ cd lab-openshift-workshop
$ git submodule update --init --recursive
```

Deploy the Spawner

```bash
$ .workshop/scripts/deploy-spawner.sh
```

Rebuild, and re-deploy custom image

```bash
$ .workshop/scripts/build-workshop.sh
```
