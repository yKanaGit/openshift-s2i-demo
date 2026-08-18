# OpenShift S2I + OpenShift Pipelines Demo

A small Node.js web application for demonstrating **OpenShift Source-to-Image (S2I)** and automatic application updates with **Red Hat OpenShift Pipelines**.

This repository intentionally contains **no Containerfile / Dockerfile**.

## Demo story

### First deployment

```text
GitHub source code
      ↓
OpenShift Developer Console
      ↓
Import from Git
      ↓
Node.js S2I Builder
      ↓
BuildConfig / ImageStream
      ↓
Deployment / Service / Route
      ↓
Application running
```

### Subsequent updates

```text
GitHub Push
      ↓
Pipelines as Code
      ↓
OpenShift PipelineRun
      ↓
Existing S2I BuildConfig starts a new build
      ↓
ImageStream is updated
      ↓
Deployment automatically rolls out the new image
      ↓
Updated application
```

## Repository structure

```text
.
├── .tekton/
│   └── pipelinerun.yaml
├── package.json
├── server.js
├── public/
│   ├── index.html
│   └── style.css
└── README.md
```

## 1. First deployment with S2I

For the demo, use the OpenShift Developer Console.

1. Switch to **Developer** perspective.
2. Select **+Add**.
3. Choose **Import from Git**.
4. Enter:

   `https://github.com/yKanaGit/openshift-s2i-demo`

5. Select a supported **Node.js** builder image.
6. Set the application name to **openshift-s2i-demo**.
7. Enable **Create a route to the application**.
8. Click **Create**.

This demonstrates that OpenShift can build and run the application directly from source code without a Containerfile.

## 2. Pipeline for subsequent Git updates

The Pipelines as Code definition is stored at:

```text
.tekton/pipelinerun.yaml
```

It listens for pushes to `main` and performs two visible steps:

```text
s2i-build → verify-rollout
```

### s2i-build

The Pipeline reuses the **BuildConfig created by the first S2I deployment**:

```bash
oc start-build openshift-s2i-demo --commit=<Git commit> --follow --wait
```

This means the initial deployment and CI build use the same S2I build mechanism.

### verify-rollout

After the S2I build completes, the Pipeline waits until the Deployment has rolled out the new image.

## 3. Prerequisites for automatic Pipeline execution

Before the demo, configure **Pipelines as Code** for this GitHub repository.

Required items:

- Red Hat OpenShift Pipelines Operator installed
- Pipelines as Code enabled
- This GitHub repository connected to Pipelines as Code
- ServiceAccount `pipeline` allowed to start the BuildConfig and read the Deployment

The GitHub/Pipelines as Code connection is setup work and does not need to be shown during the demo.

## 4. Demo procedure

### Part A: Show S2I

Start with no application resources in the demo project.

Use **Import from Git** to create `openshift-s2i-demo` and wait for the S2I build to complete.

Open the Route and show the running application.

### Part B: Show CI/CD

Edit a visible value in:

```text
public/index.html
```

Commit the change to the `main` branch.

Then open:

```text
Developer → Pipelines → PipelineRuns
```

Show the PipelineRun progressing through:

```text
s2i-build → verify-rollout
```

Finally refresh the application Route and show that the GitHub change has been deployed.

## Key message for the demo

```text
Initial deployment:
Source → S2I → Container Image → Application

After CI/CD setup:
Git Push → Pipeline → S2I rebuild → New Image → Automatic rollout
```

The application build stays based on S2I throughout the demonstration.

## Local run

```bash
npm start
```

Then open `http://localhost:8080`.

Health endpoint: `/health`
