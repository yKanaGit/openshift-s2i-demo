# OpenShift S2I + CI/CD Demo

A small Node.js web application designed for demonstrating **OpenShift Source-to-Image (S2I)** and a simple **CI/CD flow with OpenShift Pipelines**.

The repository intentionally contains **no Containerfile / Dockerfile**. OpenShift can detect the Node.js application from `package.json`, use a Node.js builder image, build a runnable container image, and deploy it.

## 1. Deploy the application with S2I

### OpenShift Developer Console

1. Switch to **Developer** perspective.
2. Select **+Add**.
3. Choose **Import from Git**.
4. Enter this repository URL:

   `https://github.com/yKanaGit/openshift-s2i-demo`

5. Select a supported **Node.js** builder image (Node.js 20 or newer is recommended).
6. Set the application name to **openshift-s2i-demo**.
7. Ensure the application port is `8080` if prompted.
8. Enable **Create a route to the application**.
9. Click **Create**.

OpenShift will perform the S2I build, create the application image, deploy it, and expose it through a Route.

### CLI example

```bash
oc new-app nodejs:20~https://github.com/yKanaGit/openshift-s2i-demo.git --name=openshift-s2i-demo
oc expose service/openshift-s2i-demo
oc get route openshift-s2i-demo
```

The exact ImageStream tag available can vary by OpenShift version and cluster configuration. If `nodejs:20` is unavailable, select an available Node.js builder image from the Developer Console.

## 2. Add the CI/CD demo

This repository contains a Pipelines as Code definition at:

`.tekton/pipelinerun.yaml`

The demo flow is intentionally simple:

```text
GitHub Push
    ↓
Source change detected
    ↓
OpenShift S2I Build
    ↓
New container image
    ↓
Deployment rollout
    ↓
Updated application
```

### Prerequisites

- Red Hat OpenShift Pipelines Operator installed
- Pipelines as Code enabled/configured for the cluster
- This GitHub repository connected to Pipelines as Code
- An existing S2I application named `openshift-s2i-demo`

The PipelineRun uses the `pipeline` ServiceAccount. It must be able to start BuildConfig builds and restart/read the application Deployment in the demo namespace.

For a demo namespace, grant the Pipeline ServiceAccount the permissions required by your cluster policy. For example, an administrator can assign an appropriate namespace-scoped role to `system:serviceaccount:<project>:pipeline`.

## 3. CI/CD demo procedure

1. Open the application Route and show the current page.
2. Edit a visible value in `public/index.html` on GitHub, such as the dashboard title.
3. Commit the change to `main`.
4. Open **Developer → Pipelines → PipelineRuns** in OpenShift.
5. Show the PipelineRun progressing through:

```text
source-change → s2i-build → deploy
```

6. Refresh the Route.
7. Show that the application contains the GitHub change.

This demonstrates the complete flow:

**Source change → CI build → container image → CD deployment**

## What the S2I demo shows

```text
Git source code
      ↓
Node.js S2I Builder
      ↓
BuildConfig / Build
      ↓
Container Image
      ↓
Deployment
      ↓
Service
      ↓
Route
      ↓
Web Application
```

No Containerfile is required for this demo.

## Local run

```bash
npm start
```

Then open `http://localhost:8080`.
