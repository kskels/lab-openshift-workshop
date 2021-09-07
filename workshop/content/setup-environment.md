This lab uses OpenShift Container Platform 4.8 cluster that is provisioned with Red Hat Product Demo System. The cluster is setup temporary and will be decommissioned soon after the workshop is concluded.

There will be multiple persons using the system with unique usernames. Setup OpenShift username environment variable as you will be assigned one during the introduction of the workshop.

The workshop instructions will use `user2` but expect it to be unique per participant.

```bash
oc whoami
export OPENSHIFT_USERNAME=`oc whoami`

echo ${OPENSHIFT_USERNAME}
```

Create and delete a project to make sure all necessary permissions are in place

```bash
oc new-project ${OPENSHIFT_USERNAME}-proj0
oc get projects

oc delete project ${OPENSHIFT_USERNAME}-proj0
oc get projects
```
