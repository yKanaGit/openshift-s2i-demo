# OpenShift S2I + GitOps Demo

A small Node.js web application designed for demonstrating **OpenShift Source-to-Image (S2I)** and **OpenShift GitOps / Argo CD**.

This repository intentionally contains **no Containerfile / Dockerfile**. OpenShift can detect the Node.js application from `package.json`, use a Node.js builder image, and build a runnable container image.

## Repository structure

```text
.
├── package.json
├── server.js
├── public/
├── manifests/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── route.yaml
└── gitops/
    └── application.yaml
```

## 1. Create the S2I build

Create the namespace first:

```bash
oc new-project openshift-s2i-demo
```

Then create the S2I build resources from this GitHub repository:

```bash
oc new-build nodejs:20~https://github.com/yKanaGit/openshift-s2i-demo.git \
  --name=openshift-s2i-demo
```

If `nodejs:20` is not available on your cluster, select an available Node.js builder image.

Start the first build and wait for it to complete:

```bash
oc start-build openshift-s2i-demo --follow
```

This creates the application image in the namespace ImageStream:

```text
openshift-s2i-demo:latest
```

## 2. Deploy with OpenShift GitOps / Argo CD

The Argo CD Application definition is stored in:

```text
gitops/application.yaml
```

Apply it once:

```bash
oc apply -f https://raw.githubusercontent.com/yKanaGit/openshift-s2i-demo/main/gitops/application.yaml
```

The Application watches:

```text
Repository: https://github.com/yKanaGit/openshift-s2i-demo.git
Branch:     main
Path:       manifests
Namespace:  openshift-s2i-demo
```

Automatic sync, prune, and self-heal are enabled.

Argo CD manages:

- Deployment
- Service
- Route

## 3. GitOps demo

A simple GitOps demo is to change the replica count in:

```text
manifests/deployment.yaml
```

For example:

```yaml
replicas: 1
```

Change it to:

```yaml
replicas: 3
```

Commit the change to `main`.

Argo CD detects the Git change and automatically synchronizes the OpenShift resources.

The demo flow is:

```text
GitHub
  ↓
Git manifest changed
  ↓
Argo CD detects OutOfSync
  ↓
Automatic Sync
  ↓
OpenShift Deployment updated
  ↓
Synced / Healthy
```

## 4. Self-heal demo

With `selfHeal: true`, Git remains the source of truth.

For example, if Git declares three replicas but somebody manually changes the Deployment to one replica:

```bash
oc scale deployment/openshift-s2i-demo --replicas=1
```

Argo CD detects the drift and restores the Deployment to the replica count declared in Git.

## S2I and GitOps roles

```text
Application source code
        ↓
OpenShift S2I
        ↓
Container Image

Git manifests
        ↓
Argo CD
        ↓
Deployment / Service / Route
```

S2I handles **building the application image**. Argo CD handles **maintaining the desired deployment state**.

## Local run

```bash
npm start
```

Then open `http://localhost:8080`.

Health endpoint:

```text
/health
```
