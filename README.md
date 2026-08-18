# OpenShift S2I + GitHub Webhook Demo

A small Node.js web application for demonstrating **OpenShift Source-to-Image (S2I)** and automatic application updates after a GitHub push.

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
GitHub Webhook
      ↓
Existing BuildConfig starts a new S2I build
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

## 2. Configure automatic rebuilds with a GitHub webhook

After the first S2I deployment, the application has a BuildConfig named:

```text
openshift-s2i-demo
```

Confirm that your user can manage the build resources:

```bash
oc auth can-i update buildconfigs
oc auth can-i create builds
```

Both should return `yes`.

### Check the GitHub webhook trigger

```bash
oc get bc openshift-s2i-demo -o yaml
```

Look under `spec.triggers` for a trigger of type `GitHub`.

If a GitHub trigger does not exist, add one:

```bash
oc set triggers bc/openshift-s2i-demo --from-github
```

### Get the webhook URL

Run:

```bash
oc describe bc openshift-s2i-demo
```

Find the **GitHub Webhook** URL in the output.

If the URL shows `<secret>`, inspect the BuildConfig to determine whether the webhook uses an inline secret or a Secret reference:

```bash
oc get bc openshift-s2i-demo -o yaml
```

Use the actual secret value in the webhook URL. Do not enter the literal string `<secret>` in GitHub.

### Register the webhook in GitHub

In this repository, open:

```text
Settings → Webhooks → Add webhook
```

Configure:

```text
Payload URL:   <OpenShift GitHub webhook URL>
Content type:  application/json
Secret:        leave empty
Events:        Just the push event
Active:        enabled
```

This setup is done once before the demo.

## 3. Demo procedure

### Part A: Show S2I

Start with no application resources in the demo project.

Use **Import from Git** to create `openshift-s2i-demo` and wait for the S2I build to complete.

Open the Route and show the running application.

### Part B: Show automatic updates

Edit a visible value in:

```text
public/index.html
```

Commit the change to the `main` branch.

The push automatically triggers:

```text
GitHub Push
      ↓
Webhook
      ↓
S2I Build
      ↓
New container image
      ↓
Automatic rollout
```

Watch the new build:

```bash
oc get builds -w
```

Watch the new Pod rollout:

```bash
oc get pods -w
```

Finally, refresh the application Route and confirm that the GitHub change is visible.

## Key message for the demo

```text
Initial deployment:
Source → S2I → Container Image → Application

After webhook setup:
Git Push → S2I rebuild → New Image → Automatic rollout
```

The developer only changes application source code in GitHub. OpenShift handles the rebuild and rollout automatically.

## Local run

```bash
npm start
```

Then open `http://localhost:8080`.

Health endpoint: `/health`
