# $\color{yellow}{\text{Local Image Registry Setup (OpenShift) }}$

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

#  $\color{yellow}{\text{Replace Ingress with Route (The Deployment Phase)}}$
## 1.Create the apps from the local images
```bash
oc new-app mood --name=mood-app
oc new-app reactions --name=reactions-app
oc new-app frontend --name=frontend-app
```
## 2.Create the Route for the Frontend
```bash
oc expose svc/frontend-app
```

# $\color{yellow}{\text{Connection Between Mood and Reaction}}
## 1.Connect Frontend to Mood/Reaction Service
```bash
oc set env deployment/frontend-app MOOD_API_URL=http://mood-app:8080

oc set env deployment/frontend-app REACTION_API_URL=http://reactions-app:8080
```
## 2.Check that the environment variables were applied correctly
```bash
oc set env deployment/frontend-app --list
```

# $\color{yellow}{\text{CHPA, Quotas, and Limits}}
## 1.Resource Quota (Project Level)
```bash
oc create quota graduation-quota --hard=cpu=4,memory=4Gi,pods=20
```
## 2.Limit Range (Pod Level)
### 2.1. Open Notepad and paste this code then save:
```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: core-limits
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 200m
      memory: 256Mi
    type: Container
```
### 2.2. Run this command in your CMD:
```bash
oc apply -f D:\limits.yaml
```
## 3.HPA (Scaling)
```bash
oc autoscale deployment/frontend-app --min=1 --max=3 --cpu-percent=80
```
## 4.Check
```bash
oc get quota,limitrange,hpa
```
