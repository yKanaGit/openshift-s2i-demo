# OpenShift S2I + GitHub Webhook Demo

A small Node.js web application for demonstrating **OpenShift Source-to-Image (S2I)** and automatic application updates from a GitHub push.

This repository intentionally contains **no Containerfile / Dockerfile**.

The demo flow is:

```text
GitHub source code
      ↓
GitHub Webhook
      ↓
OpenShift BuildConfig
      ↓
S2I Build
      ↓
ImageStream updated
      ↓
Deployment automatically rolls out
      ↓
Updated application
```

## Repository structure

```text
.
├── package.json
├── server.js
├── public/
│   ├── index.html
│   └── style.css
└── README.md
```

## 1. Create the application with S2I

Create a project:

```bash
oc new-project openshift-s2i-demo
```

Create the application directly from GitHub:

```bash
oc new-app nodejs:20~https://github.com/yKanaGit/openshift-s2i-demo.git \
  --name=openshift-s2i-demo
```

If `nodejs:20` is not available on your cluster, select an available Node.js builder image from the OpenShift Developer Console.

Expose the application:

```bash
oc expose service/openshift-s2i-demo
oc get route openshift-s2i-demo
```

Open the Route and confirm the application is running.

## 2. Confirm the S2I resources

The S2I application normally creates resources such as:

```text
BuildConfig
ImageStream
Deployment
Service
Route
```

Check the build:

```bash
oc get builds
```

Check the image stream:

```bash
oc get imagestream openshift-s2i-demo
```

## 3. Get the GitHub webhook URL

Run:

```bash
oc describe bc openshift-s2i-demo
```

In the output, find the **GitHub Webhook** URL.

You can also inspect the trigger configuration with:

```bash
oc get bc openshift-s2i-demo -o yaml
```

## 4. Configure the webhook in GitHub

In this GitHub repository, open:

```text
Settings → Webhooks → Add webhook
```

Configure:

```text
Payload URL:   <GitHub webhook URL from the BuildConfig>
Content type:  application/json
Events:        Just the push event
Active:        Enabled
```

Save the webhook.

## 5. Automatic update demo

Open the current application Route first.

Then edit a visible value in:

```text
public/index.html
```

For example, change a heading or version label and commit the change to the `main` branch.

The GitHub push triggers the following automatically:

```text
Commit to GitHub
      ↓
Webhook sent to OpenShift
      ↓
New S2I Build starts
      ↓
New application image is created
      ↓
ImageStream tag changes
      ↓
Deployment gets the new image
      ↓
New Pod starts
```

Watch the new build:

```bash
oc get builds -w
```

Watch the rollout:

```bash
oc get pods -w
```

Then refresh the application Route and confirm that the GitHub change is visible.

## What to explain during the demo

The key point is that the developer only changes application source code in GitHub.

OpenShift then handles:

1. Detecting the Git push through the webhook.
2. Rebuilding the application with S2I.
3. Creating a new container image.
4. Updating the running application with that image.

No Containerfile is required for this demo.

## Local run

```bash
npm start
```

Then open:

```text
http://localhost:8080
```

Health endpoint:

```text
/health
```
