# OpenShift GitOps Lab

## Check if GitOps Operator is installed:

### Get Routes:
```
oc get -n openshift-gitops routes openshift-gitops-server
```
Output will be somehting like: 
> NAME                      HOST/PORT                                                             PATH   SERVICES                  PORT    TERMINATION            WILDCARD<br>
> openshift-gitops-server   openshift-gitops-server-openshift-gitops.apps.hub.5g-deployment.lab          openshift-gitops-server   https   passthrough/Redirect   None

### Get Secret: 
```
oc get -n openshift-gitops secrets openshift-gitops-cluster -o jsonpath='{.data.admin\.password}' | base64 -d
```
output will be:
> bWHpNqcBdt6UDXPMQo1Z3xAYGKE8J4mS[root@hypervisor ~]#

### Check GitOps deployments are health:
```
oc get deployment -n openshift-gitops
```
output should show all deployments are running successfully 

### ArgoCD GUI: 

Access ArgoCD GUI using the information above. The GUI might look like the following: 

![gitops_1](images/gitops_1.png)

## Running ArgoCD Applications to demonstrate SyncWaves:

Run the following commands:
```
sudo -i
mkdir ~/gitops
cd ~/gitops
```

### Create Application:

```
cat << EOF > app1.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  annotations:
  labels:
    app.kubernetes.io/instance: ebc-multicloud-gitops-hub
  name: experiment-app1
  namespace: openshift-gitops
spec:
  destination:
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    directory:
      recurse: true
    path: manifests/set1
    repoURL: https://github.com/git-shassan/ocp-gitops.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 20
EOF
```

### Run the first application:

This application will demonstrate the use of SyncWaves. The applications being called are defined with the following Sync Wave values: 

| manifest | kind | name | namespace | Phase | SyncWave | Notes |
| ------------- | ------------- |------------- | ------------- |------------- |------------- |------------- |
| ns1.yaml  | Namespace | argotest1-1 | | Sync | 200 | |
| sa1.yaml  | ServiceAccount | cli-job-sa | argotest1-1 | Sync | 201 | |
| role1.yaml  | ClusterRoleBinding | cli-job-sa-argotest1-1-rolebinding || Sync | 202 | |
| job1.yaml | Job | testjob-1-1 | argotest1-1 |Sync | 203 | This job will take 100 seconds to finish | 
| powerpod2.yaml | Namespace | argotest1-2 || Sync | 300 | This resource will have to wait for "tesetjob-1-1" to complete"
| powerpod2.yaml | ServiceAccount | cli-job-sa | argotest1-2 | Sync | 300 | |
| powerpod2.yaml | ClusterRoleBinding | cli-job-sa-argotest1-2-rolebinding | argotest1-2 | Sync | 302 | |
| powerpod2.yaml | Job | testjob1-2 | argotest1-2 | Sync | 303 | |

Before running the application, check status of currently running applications: 
```
oc get applications.argoproj.io -A
```
Run the application using following command: 
```
oc apply -f app1.yaml 
```
Now observe that the application shows up on ArgoCD's GUI: 

![gitops_2](images/gitops_2.png)

The image shows that the application hasn't synced yet. That is because all the resources haven't been successfully applied. This is because "testjob1-1" is still running and hasn't reached a "Healthy" Status (its stutus will show as "Progressing")

Take a look at testjob1-1's logs, and it will show that the counter is still running: 

![gitops_3](images/gitops_3.png)

Once the counter reaches 10, the job will complete, and now the next resource will be applied. The next resource happens to be creation of Namespace called "argotest1-2", based on SyncWave values as seen in the above table. Then the subsequenet one will be applied, and so on. 

Eventually, the application reaches a full sync status once all resources have reaached a Healthy status: 

![gitops_4](images/gitops_4.png)




