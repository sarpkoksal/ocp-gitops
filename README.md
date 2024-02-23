# ocp-gitops

The following table should be updated based on last re-organization of apps/manifests: 

# App1: 

Manifests being called by app1.yaml:
(The table has been arranged to reflect order or execution based on Phase Hooks and Sync Waves)

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

# App2: 
Manifests being called by app2.yaml:
(The table has been arranged to reflect order or execution based on Phase Hooks and Sync Waves)
| manifest | kind | name | namespace | Phase | SyncWave | Notes |
| ------------- | ------------- |------------- | ------------- |------------- |------------- |------------- |
| ns.yaml | Namespace | argotest2 | | PreSync | 1 | |
| presync1.yaml | Job | presync1 | argotest2 | PreSync | 103 | The job will take 100 seconds to complete | 
| presync2.yaml | Job | presync2 | argotest2 | PreSync | 203 | The job will run BEFORE testjob1 because of PreSync hook |  
| powerpod.yaml | Job | testjob1 | argotest2 | Sync | 103 | presync2 will never complete, so this job will never start | 

