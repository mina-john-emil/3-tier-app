# 🏗️ Local Image Registry Setup (OpenShift)

This section guides you through setting up the **ImageStreams** and **BuildConfigs** required to build and host your container images directly within the OpenShift cluster.

---

## 1. Create the GitHub Secret
First, create a secret to allow OpenShift to pull your private source code from GitHub.

### **A) Create the secret**
```bash
oc create secret generic github-token-secret --from-literal=password=<<ask me for token>> --type=kubernetes.io/basic-auth
oc annotate secret github-token-secret 'build.openshift.io/source-secret-match-uri-1=https://github.com/mina-john-emil/*'
