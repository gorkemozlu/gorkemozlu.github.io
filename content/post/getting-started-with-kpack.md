---
title: Getting Started with kpack
date: 2021-10-24
tags:
  - "kpack"
  - "build"
  - "development"
categories:
  - "kpack"
  - "build"
  - "development"

---
```kpack``` is a K8s-native Way to Build and Update Containers.
<!--more-->

## Prep.

Prepare a conformant kubernetes cluster. Deploy kpack first. Then, optionally install ```stern```

```sh
kubectl apply -f https://github.com/pivotal/kpack/releases/download/v0.4.0/release-0.4.0.yaml
```
## Step 1. 

```sh
$ kubectl apply -f secret.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub-registry-credentials
  annotations:
    kpack.io/docker: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
stringData:
  username: "username"
  password: "password"
```

## Step 2. 

```sh
$ kubectl apply -f sa.yaml
```
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dockerhub-service-account
secrets:
- name: dockerhub-registry-credentials
imagePullSecrets:
- name: dockerhub-registry-credentials
```

## Step 3. ClusterStore

```sh
$ kubectl apply -f clusterstore.yaml
```

```yaml
apiVersion: kpack.io/v1alpha1
kind: ClusterStore
metadata:
  name: default
spec:
  sources:
  - image: gcr.io/paketo-buildpacks/java
```

## Step 4. ClusterStack

```sh
$ kubectl apply -f clusterstack.yaml
```

```yaml
apiVersion: kpack.io/v1alpha1
kind: ClusterStack
metadata:
  name: base
spec:
  id: "io.buildpacks.stacks.bionic"
  buildImage:
    image: "paketobuildpacks/build:base-cnb"
  runImage:
    image: "paketobuildpacks/run:base-cnb"
```
## Step 4. Builder

```sh
$ kubectl apply -f builder.yaml
```

```yaml
apiVersion: kpack.io/v1alpha1
kind: Builder
metadata:
  name: my-builder
  namespace: default
spec:
  serviceAccount: dockerhub-service-account
  tag: index.docker.io/bgorkemozlu/builder
  stack:
    name: base
    kind: ClusterStack
  store:
    name: default
    kind: ClusterStore
  order:
  - group:
    - id: paketo-buildpacks/java
```

## Step 4. Image

```sh
$ kubectl apply -f image.yaml
```

```yaml
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
      revision: 82cb521d636b282340378d80a6307a08e3d4a4c4
```

```terminal
$ kubectl get images
NAME              LATESTIMAGE   READY
petclinic-image                 Unknown
```
```terminal
$ kubectl get pods
NAME                                READY   STATUS     RESTARTS   AGE
petclinic-image-build-1-build-pod   0/1     Init:0/6   0          15s
```
```terminal
$ stern -n default petclinic-image-build-1-build-pod
```
```terminal
$ kubectl get pods
NAME                                READY   STATUS      RESTARTS   AGE
petclinic-image-build-1-build-pod   0/1     Completed   0          9m
```
```terminal
$ kubectl get images
NAME              LATESTIMAGE                                                                                                  READY
petclinic-image   index.docker.io/bgorkemozlu/public@sha256:fadd73872877c763dce77b4df8f9b693323540657ba823fdd2eff329e9f120ab   True
```
```terminal
$ kubectl create deployment kpack-demo --image=index.docker.io/bgorkemozlu/public@sha256:fadd73872877c763dce77b4df8f9b693323540657ba823fdd2eff329e9f120ab
deployment.apps/kpack-demo created
$ kubectl expose deployment kpack-demo --port=8080
service/kpack-demo exposed
$ kubectl port-forward svc/kpack-demo 8080:8080
```