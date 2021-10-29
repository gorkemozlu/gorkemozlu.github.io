---
title: Updating images with kpack
date: 2021-10-24
tags:
  - "kpack"
  - "build"
  - "development"
categories:
  - "kpack"
  - "build"
  - "development"
thumbnail: img/kpack.png
---
Last time we built an image with kpack. This time we'll be updating the image without hassle.
<!--more-->

Earlier logs shows that revision 82cb521d636b282340378d80a6307a08e3d4a4c4 checked-out when we applied the yaml.

```sh
stern pet
+ petclinic-image-build-1-build-pod › prepare
petclinic-image-build-1-build-pod prepare Build reason(s): CONFIG
petclinic-image-build-1-build-pod prepare CONFIG:
petclinic-image-build-1-build-pod prepare       resources: {}
petclinic-image-build-1-build-pod prepare       - source: {}
petclinic-image-build-1-build-pod prepare       + source:
petclinic-image-build-1-build-pod prepare       +   git:
petclinic-image-build-1-build-pod prepare       +     revision: 82cb521d636b282340378d80a6307a08e3d4a4c4
petclinic-image-build-1-build-pod prepare       +     url: https://github.com/spring-projects/spring-petclinic
petclinic-image-build-1-build-pod prepare Loading secrets for "https://index.docker.io/v1/" from secret "dockerhub-registry-credentials"
petclinic-image-build-1-build-pod prepare Cloning "https://github.com/spring-projects/spring-petclinic" @ "82cb521d636b282340378d80a6307a08e3d4a4c4"...
petclinic-image-build-1-build-pod prepare Successfully cloned "https://github.com/spring-projects/spring-petclinic" @ "82cb521d636b282340378d80a6307a08e3d4a4c4" in path "/workspace"
```

## Step 1. Update rev-id in yaml

Now, we're going to update the image yaml with an another revision.
```sh
$ sed 's/82cb521d636b282340378d80a6307a08e3d4a4c4/01621077cbf76aa873eab64f0f0f50aaf6a04b90/g' 'image.yaml'
apiVersion: kpack.io/v1alpha1
kind: Image
metadata:
  name: petclinic-image
  namespace: default
spec:
  tag: index.docker.io/bgorkemozlu/public
  serviceAccount: dockerhub-service-account
  builder:
    name: my-builder
    kind: Builder
  source:
    git:
      url: https://github.com/spring-projects/spring-petclinic
      revision: 01621077cbf76aa873eab64f0f0f50aaf6a04b90
```
## Step 2. Update image build


```terminal
$ kubectl apply -f image.yaml
image.kpack.io/petclinic-image configured
```
Now, we see a new build pod show up.
```sh
$ kubectl get pods
NAME                                READY   STATUS      RESTARTS   AGE
kpack-demo-5698f7dd84-7rxrv         1/1     Running     0          15m
petclinic-image-build-1-build-pod   0/1     Completed   0          55m
petclinic-image-build-2-build-pod   0/1     Init:0/6    0          21s
```
It also says new revision has picked up.
```sh
stern -n default petclinic-image-build-2-build-pod
+ petclinic-image-build-2-build-pod › prepare
petclinic-image-build-2-build-pod prepare Build reason(s): COMMIT
petclinic-image-build-2-build-pod prepare COMMIT:
petclinic-image-build-2-build-pod prepare       - 82cb521d636b282340378d80a6307a08e3d4a4c4
petclinic-image-build-2-build-pod prepare       + 01621077cbf76aa873eab64f0f0f50aaf6a04b90
petclinic-image-build-2-build-pod prepare Loading secrets for "https://index.docker.io/v1/" from secret "dockerhub-registry-credentials"
petclinic-image-build-2-build-pod prepare Cloning "https://github.com/spring-projects/spring-petclinic" @ "01621077cbf76aa873eab64f0f0f50aaf6a04b90"...
```

## Step 3. Check updated image

```sh
$ kubectl get images
NAME              LATESTIMAGE                                                                                                  READY
petclinic-image   index.docker.io/bgorkemozlu/public@sha256:4d11e0180343ba1a76624028c3f16941b5671d35295f742e48bff18a5628057b   True
```
## Step 4. Deploy the updated image

```sh
$ kubectl patch deployment kpack-demo -p \
  '{"spec":{"template":{"spec":{"containers":[{"name":"public","image":"index.docker.io/bgorkemozlu/public@sha256:4d11e0180343ba1a76624028c3f16941b5671d35295f742e48bff18a5628057b"}]}}}}'
deployment.apps/kpack-demo patched
$ kubectl expose deployment kpack-demo --port=8080
service/kpack-demo exposed
$ kubectl port-forward svc/kpack-demo 8080:8080
```
