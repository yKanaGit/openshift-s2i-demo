# OpenShift S2I Demo

A small Node.js web application designed for demonstrating **OpenShift Source-to-Image (S2I)**.

The repository intentionally contains **no Containerfile / Dockerfile**. OpenShift can detect the Node.js application from `package.json`, use a Node.js builder image, build a runnable container image, and deploy it.

## Deploy from the OpenShift Developer Console

1. Switch to **Developer** perspective.
2. Select **+Add**.
3. Choose **Import from Git**.
4. Enter this repository URL:

   `https://github.com/yKanaGit/openshift-s2i-demo`

5. Select a supported **Node.js** builder image (Node.js 20 or newer is recommended).
6. Ensure the application port is `8080` if prompted.
7. Enable **Create a route to the application**.
8. Click **Create**.

OpenShift will perform the S2I build, create the application image, deploy it, and expose it through a Route.

## CLI example

```bash
oc new-app nodejs:20~https://github.com/yKanaGit/openshift-s2i-demo.git --name=s2i-demo
oc expose service/s2i-demo
oc get route s2i-demo
```

The exact ImageStream tag available can vary by OpenShift version and cluster configuration. If `nodejs:20` is unavailable, select an available Node.js builder image from the Developer Console.

## What this demo shows

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
