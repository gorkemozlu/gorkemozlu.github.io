---
title: Operations on Tanzu Community Edition
date: "2021-10-30T00:00:00+03:00"
tags:
  - "Tanzu"
  - "TCE"
categories:
  - "Tanzu"
  - "TCE"
thumbnail: img/tce-02/cluster-api.png
---
On our last post we deployed Tanzu Community Edition with various ways. Today, we'll cover more details like advantages of ClusterAPI and Tanzu packages.
<!--more-->

As you can see below, cluster-001's worker node is up ```cluster-001-md-0-7754fbbdb9-r5ttt``` and running for 59 minutes.

```terminal
❯ docker ps --format '{{.Names}},{{.RunningFor}},{{.ID}}'
cluster-001-md-0-7754fbbdb9-r5ttt,59 minutes ago,8593f398309f
cluster-001-control-plane-zk8ck,About an hour ago,e53bc1e99a37
cluster-001-lb,About an hour ago,3cc0f58783e8
mgt-cluster-md-0-67d55c7fcd-nzvzq,About an hour ago,588e6ea02b58
mgt-cluster-control-plane-dbpk2,About an hour ago,47887bd30f1f
mgt-cluster-lb,About an hour ago,dc7595716e26
```
Now, we're going to terminate that worker node.
```terminal
❯ docker stop 8593f398309f
8593f398309f
❯ docker rm 8593f398309f
8593f398309f
❯ docker ps --format '{{.Names}},{{.RunningFor}},{{.ID}}'
cluster-001-control-plane-zk8ck,About an hour ago,e53bc1e99a37
cluster-001-lb,About an hour ago,3cc0f58783e8
mgt-cluster-md-0-67d55c7fcd-nzvzq,About an hour ago,588e6ea02b58
mgt-cluster-control-plane-dbpk2,About an hour ago,47887bd30f1f
mgt-cluster-lb,About an hour ago,dc7595716e26
```
Then ClusterAPI will detect worker node is missing and will start to reconcile missing node.

```terminal
capi-system capi-controller-manager-778bd4dfb9-qvl4s manager E1029 15:38:58.523247       1 machine_controller.go:685] controllers/Machine "msg"="Unable to retrieve machine from node" "error"="no matching Machine"  "node"="cluster-001-md-0-7754fbbdb9-r5ttt"
capi-system capi-controller-manager-778bd4dfb9-qvl4s manager E1029 15:38:58.523713       1 machinehealthcheck_controller.go:480] controllers/MachineHealthCheck "msg"="Unable to retrieve machine from node" "error"="expecting one machine for node cluster-001-md-0-7754fbbdb9-r5ttt, got []"  "node"="cluster-001-md-0-7754fbbdb9-r5ttt"
capi-system capi-controller-manager-778bd4dfb9-qvl4s manager I1029 15:39:03.015782       1 machinehealthcheck_controller.go:109] controllers/MachineHealthCheck "msg"="Reconciling" "machinehealthcheck"="cluster-001" "namespace"="default" 
```

After couple minutes later, we'll be able to see a new node is placed and ready.
```
❯ docker ps --format '{{.Names}},{{.RunningFor}},{{.ID}}'
cluster-001-md-0-7754fbbdb9-8lsx5,2 minutes ago,34e1f3fd6976
cluster-001-control-plane-zk8ck,About an hour ago,e53bc1e99a37
cluster-001-lb,About an hour ago,3cc0f58783e8
mgt-cluster-md-0-67d55c7fcd-nzvzq,About an hour ago,588e6ea02b58
mgt-cluster-control-plane-dbpk2,2 hours ago,47887bd30f1f
mgt-cluster-lb,2 hours ago,dc7595716e26

❯ kubectl get nodes
NAME                                STATUS                        ROLES                  AGE     VERSION
cluster-001-control-plane-zk8ck     Ready                         control-plane,master   70m     v1.21.2+vmware.1-360497810732255795
cluster-001-md-0-7754fbbdb9-8lsx5   Ready                         <none>                 4m41s   v1.21.2+vmware.1-360497810732255795                                  
```



Tanzu Community Edition provides a simple way to deploy additional Packages to the Kubernetes Cluster.

There is a well-maintained repository that makes several components available. 

```terminal
❯ tanzu package repository add tce-repo \
  --url projects.registry.vmware.com/tce/main:0.9.1 \
  --namespace tanzu-package-repo-global

❯ tanzu package repository list --namespace tanzu-package-repo-global
/ Retrieving repositories... 
  NAME      REPOSITORY                                   STATUS               DETAILS  
  tce-repo  projects.registry.vmware.com/tce/main:0.9.1  Reconcile succeeded           
❯ tanzu package available list

/ Retrieving available packages... 
  NAME                                           DISPLAY-NAME        SHORT-DESCRIPTION                                                                                                             
  cert-manager.community.tanzu.vmware.com        cert-manager        Certificate management                                                                                                        
  contour.community.tanzu.vmware.com             Contour             An ingress controller                                                                                                         
  external-dns.community.tanzu.vmware.com        external-dns        This package provides DNS synchronization functionality.                                                                      
  fluent-bit.community.tanzu.vmware.com          fluent-bit          Fluent Bit is a fast Log Processor and Forwarder                                                                              
  gatekeeper.community.tanzu.vmware.com          gatekeeper          policy management                                                                                                             
  grafana.community.tanzu.vmware.com             grafana             Visualization and analytics software                                                                                          
  harbor.community.tanzu.vmware.com              Harbor              OCI Registry                                                                                                                  
  knative-serving.community.tanzu.vmware.com     knative-serving     Knative Serving builds on Kubernetes to support deploying and serving of applications and functions as serverless containers  
  local-path-storage.community.tanzu.vmware.com  local-path-storage  This package provides local path node storage and primarily supports RWO AccessMode.                                          
  multus-cni.community.tanzu.vmware.com          multus-cni          This package provides the ability for enabling attaching multiple network interfaces to pods in Kubernetes                    
  prometheus.community.tanzu.vmware.com          prometheus          A time series database for your metrics                                                                                       
  velero.community.tanzu.vmware.com              velero              Disaster recovery capabilities   
```
Even we can deploy storageClass as an application/package.
```terminal
kubectl get sc
No resources found
❯ tanzu package available list  local-path-storage.community.tanzu.vmware.com

/ Retrieving package versions for local-path-storage.community.tanzu.vmware.com... 
  NAME                                           VERSION  RELEASED-AT           
  local-path-storage.community.tanzu.vmware.com  0.0.19   2021-09-15T00:00:00Z  
  local-path-storage.community.tanzu.vmware.com  0.0.20   2021-09-15T00:00:00Z  
❯ tanzu package install local-path-storage --package-name local-path-storage.community.tanzu.vmware.com --version 0.0.20

/ Installing package 'local-path-storage.community.tanzu.vmware.com' 
| Getting namespace 'default' 
| Getting package metadata for 'local-path-storage.community.tanzu.vmware.com' 
| Creating service account 'local-path-storage-default-sa' 
| Creating cluster admin role 'local-path-storage-default-cluster-role' 
| Creating cluster role binding 'local-path-storage-default-cluster-rolebinding' 
- Creating package resource 
- Package install status: Reconciling 


 Added installed package 'local-path-storage' in namespace 'default'
❯  kubectl get sc
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  19s
```

It's possible to modify an application's default configuration.
```terminal
❯ cat <<EOF > prom.yaml
prometheus:
  pvc:
    storageClassName: local-path
    storage: 1Gi
EOF
❯ tanzu package available list  prometheus.community.tanzu.vmware.com

/ Retrieving package versions for prometheus.community.tanzu.vmware.com... 
  NAME                                   VERSION  RELEASED-AT           
  prometheus.community.tanzu.vmware.com  2.27.0   2021-05-12T18:00:00Z

❯ tanzu package install prometheus --package-name prometheus.community.tanzu.vmware.com --version 2.27.0 -f prom.yaml

/ Installing package 'prometheus.community.tanzu.vmware.com' 
| Getting namespace 'default' 
| Getting package metadata for 'prometheus.community.tanzu.vmware.com' 
| Creating service account 'prometheus-default-sa' 
| Creating cluster admin role 'prometheus-default-cluster-role' 
| Creating cluster role binding 'prometheus-default-cluster-rolebinding' 
| Creating secret 'prometheus-default-values' 
- Creating package resource 
/ Package install status: Reconciling 

 Added installed package 'prometheus' in namespace 'default'

❯ kubectl get pods,svc -n prometheus
NAME                                                 READY   STATUS    RESTARTS   AGE
pod/alertmanager-6877f5989c-2stqd                    1/1     Running   0          2m49s
pod/prometheus-cadvisor-8q6kl                        1/1     Running   0          2m50s
pod/prometheus-kube-state-metrics-7db8755b58-hzqsd   1/1     Running   0          2m49s
pod/prometheus-node-exporter-fvfjh                   1/1     Running   0          2m49s
pod/prometheus-node-exporter-krt97                   1/1     Running   0          2m49s
pod/prometheus-pushgateway-d46f4f587-592qw           1/1     Running   0          2m49s
pod/prometheus-server-657c5594df-4ggss               2/2     Running   0          2m49s

NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/alertmanager                    ClusterIP   100.69.3.87      <none>        80/TCP          2m50s
service/prometheus-kube-state-metrics   ClusterIP   None             <none>        80/TCP,81/TCP   2m50s
service/prometheus-node-exporter        ClusterIP   100.64.148.41    <none>        9100/TCP        2m49s
service/prometheus-pushgateway          ClusterIP   100.71.241.176   <none>        9091/TCP        2m50s
service/prometheus-server               ClusterIP   100.69.133.10    <none>        80/TCP          2m49s
```


