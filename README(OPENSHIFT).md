# 🏗️ Local Image Registry Setup (OpenShift)

This section guides you through setting up the **ImageStreams** and **BuildConfigs** required to build and host your container images directly within the OpenShift cluster.

---

## 1. Create the GitHub Secret
First, create a secret to allow OpenShift to pull your private source code from GitHub.

```bash
oc create secret generic github-token-secret --from-literal=password=<<ask me for token>> --type=kubernetes.io/basic-auth
oc annotate secret github-token-secret 'build.openshift.io/source-secret-match-uri-1=https://github.com/mina-john-emil/*'
```
## 2. The Build Configs
Create a file named all-builds.yaml and paste this. It handles all three parts of your app at once.

```bash
apiVersion: v1
kind: List
items:
- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: mood
- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: reactions
- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    name: frontend
- apiVersion: build.openshift.io/v1
  kind: BuildConfig
  metadata:
    name: mood-build
  spec:
    source:
      git: {uri: "https://github.com/mina-john-emil/3-tier-app.git"}
      contextDir: "backend/mood"
      sourceSecret: {name: "github-token-secret"}
    strategy:
      type: Docker
    output:
      to: {kind: ImageStreamTag, name: "mood:latest"}
- apiVersion: build.openshift.io/v1
  kind: BuildConfig
  metadata:
    name: reactions-build
  spec:
    source:
      git: {uri: "https://github.com/mina-john-emil/3-tier-app.git"}
      contextDir: "backend/reactions"
      sourceSecret: {name: "github-token-secret"}
    strategy:
      type: Docker
    output:
      to: {kind: ImageStreamTag, name: "reactions:latest"}
- apiVersion: build.openshift.io/v1
  kind: BuildConfig
  metadata:
    name: frontend-build
  spec:
    source:
      git: {uri: "https://github.com/mina-john-emil/3-tier-app.git"}
      contextDir: "frontend"
      sourceSecret: {name: "github-token-secret"}
    strategy:
      type: Docker
    output:
      to: {kind: ImageStreamTag, name: "frontend:latest"}

```
## 3.Apply and Start
```bash
oc apply -f all-builds.yaml
oc start-build mood-build
oc start-build reactions-build
oc start-build frontend-build
```
