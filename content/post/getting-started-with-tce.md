---
title: Getting Started with Tanzu Community Edition
date: 2021-10-29
tags:
  - "Tanzu"
  - "TCE"
categories:
  - "Tanzu"
  - "TCE"
thumbnail: img/tce-01/tce-logo.png
---
```Tanzu Community Edition``` is a full-featured, easy-to-manage Kubernetes platform for learners and users, especially those working in small-scale or preproduction environments.
<!--more-->

VMware Tanzu Community Edition is a full-featured, easy to manage Kubernetes platform for learners and users. It is a freely available, community supported, open source distribution of VMware Tanzu that can be installed and configured in minutes on your local workstation or your favorite cloud.

It includes a package repository for installing and updating components of your platform.

![image info](/img/tce-01/tce-architecture.png)
Within Tanzu Community edition, there are two methods to building Kubernetes centric platforms.

* Management + Workload Clusters :
  * This method involves provisioning a management Kubernetes Cluster and using that cluster to build more Kubernetes workload Clusters. The management cluster helps in providing/storing authentication details of workload clusters via Cluster API.
* Standalone Clusters : 
   * These are Kubernetes clusters built without a management cluster directly on the infrastructure provide of choice. Once the Standalone Cluster is built, storing and retrieving access to the Kubernetes cluster is deferred to the admin of the cluster.

You can get the latest release from the [github page](https://github.com/vmware-tanzu/community-edition/releases)

## Standalone Cluster
Once you download and extract the package you can follow below lines to create a simple standalone cluster.

```terminal
./install.sh
tanzu standalone-cluster create --ui
```
For local quick development purposes, choose docker and give cluster a name, you can leave the default network configuration alone.

![image info](/img/tce-01/tce-std-01.png)

You can either hit the deploy button or use CLI command instead.

![image info](/img/tce-01/tce-std-02.png)

![image info](/img/tce-01/tce-std-03.png)

Once it finishes, you cluster is ready and you can go ahead and execute the ```tanzu``` or ```kubectl``` commands.

```terminal
❯ kubectl get nodes
NAME                              STATUS   ROLES                  AGE     VERSION
cluster01-control-plane-mf97d     Ready    control-plane,master   7m28s   v1.21.2+vmware.1-360497810732255795
cluster01-md-0-5cccffd8d4-8gw69   Ready    <none>                 7m2s    v1.21.2+vmware.1-360497810732255795

❯ kubectl get ns
NAME                        STATUS   AGE
default                     Active   7m54s
kube-node-lease             Active   7m55s
kube-public                 Active   7m55s
kube-system                 Active   7m55s
tanzu-package-repo-global   Active   7m22s
tkg-system                  Active   7m30s
tkg-system-public           Active   7m42s
tkr-system                  Active   7m42s
```

You can delete your standalone cluster via below command.

```terminal
❯ tanzu standalone-cluster delete cluster01
Loading bootstrap cluster config for standalone cluster at '/Users/gorkem/.config/tanzu/tkg/clusterconfigs/cluster01.yaml'
Deleting standalone cluster 'cluster01'. Are you sure? [y/N]: y

loading cluster config file at /Users/gorkem/.config/tanzu/tkg/clusterconfigs/cluster01.yaml
Setting up cleanup cluster...
Installing providers to cleanup cluster...
Moving all Cluster API objects from bootstrap cluster to standalone cluster...
Waiting for the Cluster API objects to be ready after restore ...
Deleting standalone cluster...
Standalone cluster 'cluster01' deleted.
Deleting the standalone cluster context from the kubeconfig file '/Users/gorkem/.kube/config'
warning: this removed your active context, use "kubectl config use-context" to select a different one

Standalone cluster deleted!
Removing temporary bootstrap cluster config for standalone cluster at '/Users/gorkem/.config/tanzu/tkg/configs/cluster01_ClusterConfig'
no bootstrap cluster config found - skipping
Removing temporary UI bootstrap cluster config for standalone cluster at '/Users/gorkem/.config/tanzu/clusterconfigs/cluster01.yaml'
no UI bootstrap cluster config found - skipping
```

## Management Cluster

As mentioned before management cluster provides a control plane layer for keeping regular clusters at desired state.

You can follow below commands to deployment management cluster.

```terminal
tanzu management-cluster create --ui
```

![image info](/img/tce-01/tce-std-01.png)

For local quick development purposes, choose docker and give the management cluster a name, you can leave the default network configuration alone.

![image info](/img/tce-01/tce-mgt-01.png)

![image info](/img/tce-01/tce-mgt-02.png)

Since we deploy the management cluster, we can use ```tanzu``` and ```kubectl``` commands.

```terminal
❯ tanzu management-cluster get

  NAME         NAMESPACE   STATUS   CONTROLPLANE  WORKERS  KUBERNETES        ROLES       
  mgt-cluster  tkg-system  running  1/1           1/1      v1.21.2+vmware.1  management  


Details:

NAME                                                            READY  SEVERITY  REASON  SINCE  MESSAGE
/mgt-cluster                                                    True                     12m           
├─ClusterInfrastructure - DockerCluster/mgt-cluster             True                     12m           
├─ControlPlane - KubeadmControlPlane/mgt-cluster-control-plane  True                     12m           
│ └─Machine/mgt-cluster-control-plane-dbpk2                     True                     12m           
└─Workers                                                                                              
  └─MachineDeployment/mgt-cluster-md-0                                                                 
    └─Machine/mgt-cluster-md-0-67d55c7fcd-nzvzq                 True                     12m           


Providers:

  NAMESPACE                          NAME                   TYPE                    PROVIDERNAME  VERSION  WATCHNAMESPACE  
  capd-system                        infrastructure-docker  InfrastructureProvider  docker        v0.3.23                  
  capi-kubeadm-bootstrap-system      bootstrap-kubeadm      BootstrapProvider       kubeadm       v0.3.23                  
  capi-kubeadm-control-plane-system  control-plane-kubeadm  ControlPlaneProvider    kubeadm       v0.3.23                  
  capi-system                        cluster-api            CoreProvider            cluster-api   v0.3.23                  
```

```terminal
❯ tanzu management-cluster kubeconfig get mgt-cluster --admin
Credentials of cluster 'mgt-cluster' have been saved 
You can now access the cluster by running 'kubectl config use-context mgt-cluster-admin@mgt-cluster'

❯ kubectl config use-context mgt-cluster-admin@mgt-cluster
Switched to context "mgt-cluster-admin@mgt-cluster".

❯ kubectl get nodes
NAME                                STATUS   ROLES                  AGE   VERSION
mgt-cluster-control-plane-dbpk2     Ready    control-plane,master   18m   v1.21.2+vmware.1-360497810732255795
mgt-cluster-md-0-67d55c7fcd-nzvzq   Ready    <none>                 17m   v1.21.2+vmware.1-360497810732255795
```

We'll deploy a cluster via management cluster. Cluster API will handle the provisioning and storing the authentication details.

```terminal
❯ tanzu cluster create cluster-001 --plan dev
Workload cluster 'cluster-001' created

❯ tanzu cluster kubeconfig get cluster-001 --admin
Credentials of cluster 'cluster-001' have been saved 
You can now access the cluster by running 'kubectl config use-context cluster-001-admin@cluster-001'

❯ kubectl config use-context cluster-001-admin@cluster-001
Switched to context "cluster-001-admin@cluster-001".

❯ kubectl get nodes
NAME                                STATUS   ROLES                  AGE     VERSION
cluster-001-control-plane-zk8ck     Ready    control-plane,master   8m5s    v1.21.2+vmware.1-360497810732255795
cluster-001-md-0-7754fbbdb9-r5ttt   Ready    <none>                 7m34s   v1.21.2+vmware.1-360497810732255795
```

```sh
❯ docker ps --format '{{.Names}},{{.RunningFor}}'
cluster-001-md-0-7754fbbdb9-r5ttt,12 minutes ago
cluster-001-control-plane-zk8ck,13 minutes ago
cluster-001-lb,13 minutes ago
mgt-cluster-md-0-67d55c7fcd-nzvzq,33 minutes ago
mgt-cluster-control-plane-dbpk2,34 minutes ago
mgt-cluster-lb,34 minutes ago
```

We'll cover more details on next post.