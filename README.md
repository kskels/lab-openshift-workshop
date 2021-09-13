OpenShift Workshop
==================


This is a content repository for an interactive workshop that helps users to get familiar with *Kubernetes* and *OpenShift Container Platform 4* core concepts. The workshop focuses on declarative application deployment using `YAML` manifests.

See [YouTube instructions video](https://www.youtube.com/watch?v=HxBfCRGCvyc&t=251s) for information on *OpenShift Homeroom* learning environment and how to deploy this lab onto your own *OpenShift 4* cluster.

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
