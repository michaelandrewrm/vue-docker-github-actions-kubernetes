| Concept  | What it means                                                           |
| -------- | ----------------------------------------------------------------------- |
| Workflow | Automation pipeline (written in YAML) triggered by events               |
| Event    | Something that triggers a workflow (push, pull_request, schedule, etc.) |
| Job      | A group of steps that run on the same runner (machine)                  |
| Step     | A single task — usually running a command or action                     |
| Action   | A reusable piece of code (like a mini app)                              |
| Runner   | A server that executes the jobs (GitHub-hosted or self-hosted)          |

Create a `YAML` file:

```
.github/workflows/
```

```yml
# File: .github/workflows/hello-world.yml

name: Hello World Workflow

# When anyone pushes code it will run a job on a fresh Ubuntu server that echoes a message.
on: [push]

jobs:
  say_hello:
    runs-on: ubuntu-latest
    steps:
      - name: Print a message
        run: echo "Hello, GitHub Actions!"
```

### 🔥 Part 1: Triggers (aka "What kicks off the workflow")

In GitHub Actions, triggers are defined under the on: keyword.
This controls when your workflow runs.

Here are the main types of triggers:

| Feature                | Description                                                            |
| ---------------------- | ---------------------------------------------------------------------- |
| Triggers:              | push, pull_request, schedule (cron), workflow_dispatch (manual button) |
| Jobs & Steps           | Multi-job workflows, dependency between jobs                           |
| Matrix Builds          | Test across many versions (like Node.js 14, 16, 18)                    |
| Using Prebuilt Actions | From GitHub Marketplace                                                |
| Writing Custom Actions | In JavaScript or Docker                                                |
| Secrets & Security     | Managing sensitive info (API keys, tokens)                             |
| Deployments            | Automate deploys to AWS, Azure, Vercel, Netlify, etc.                  |
| Artifacts              | Upload/download build results between jobs                             |
| Caching                | Speeding up workflows with cache                                       |
| Self-Hosted Runners    | Custom runners for more power/control                                  |

Examples:
Listen for push only on main branch:

```yml
on:
  push:
    branches:
      - main
```

Manual trigger (very important for controlled deployments):

```yml
on:
  workflow_dispatch:
```

Cron schedule (run every day at midnight UTC):
Cron syntax: `min` `hour` `day` `month` `weekday`

```yml
on:
  schedule:
    - cron: '0 0 * * *'
```

### 🔥 Part 2: Jobs (aka "How you organize the work")

A job is a collection of steps that run in the same environment (called a runner).

Basic job structure:

```yml
jobs:
  build_job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

Important concepts inside jobs:

- runs-on: OS of the runner (ubuntu-latest, windows-latest, macos-latest)
- steps: Series of tasks (commands or using prebuilt actions)
- Multiple jobs can run in parallel or with dependencies

Job dependencies (needs: keyword):

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building app..."

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "Deploying app..."
```

`deploy` waits until build finishes successfully!

### 🔥 Part 3: Actions (aka "Reusable bricks of work")

Actions are prebuilt tasks you can plug into your workflows.

Example using a community action: (checking out your code)

```yml
steps:
  - name: Checkout repo
  - uses: actions/checkout@v4
```

- **uses**: means you’re using someone else's prebuilt GitHub Action.
- You can find actions in the [GitHub Marketplace](https://github.com/marketplace?type=actions).

Actions can be:

- Official (built by GitHub — like actions/checkout)
- Third-party (like JamesIves/github-pages-deploy-action for auto-deploying pages)
- Your own custom (you can write custom actions in JavaScript or Docker if needed)

You can also chain actions together with normal run: commands.

### Artifacts (Share build results between jobs)

```yml
# In build job
- name: Build static site
  run: npm run build

- name: Upload build artifact
  uses: actions/upload-artifact@v4
  with:
    name: build-folder
    path: ./dist

# In deploy job
- name: Download build artifact
  uses: actions/download-artifact@v4
  with:
    name: build-folder
    path: ./dist
```

### Matrix Builds (Run multiple configs in parallel)

```yml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    name: Test on Node ${{ matrix.node-version }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm install
      - run: npm test
```

### Managing Secrets

1. Navigate to your GitHub repository.
2. Go to **Settings** > **Secrets and variables** > **Actions**.
3. Click New repository secret and add secrets such as:
   - DOCKER_USERNAME
   - DOCKER_PASSWORD
   - KUBECONFIG

In your workflow, reference these secrets using the secrets context:

```yml
env:
  API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
```

### Project structure

```java
vue-app/
├── Dockerfile
├── .dockerignore
├── .github/
│   └── workflows/
│       └── deploy.yml
├── public/
├── src/
└── package.json

```

### Dockerfile

```
# Build phase
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Serve phase
FROM nginx:stable-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

### nginx.conf

```
server {
  listen 80;
  server_name localhost;

  location / {
    root /usr/share/nginx/html;
    index index.html;
    try_files $uri $uri/ /index.html;
  }
}

```

### .dockerignore

```
node_modules
dist
.git
```

### .github/workflows/deploy.yml

```yml
name: Vue CI/CD to Kubernetes

on:
  push:
    branches:
      - main

env:
  IMAGE_NAME: vue-app
  IMAGE_TAG: ${{ github.sha }}
  REGISTRY: docker.io
  DEPLOYMENT_NAME: vue-deployment
  NAMESPACE: default

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build and Push Docker Image
        run: |
          docker build -t $REGISTRY/${{ secrets.DOCKER_USERNAME }}/$IMAGE_NAME:$IMAGE_TAG .
          docker push $REGISTRY/${{ secrets.DOCKER_USERNAME }}/$IMAGE_NAME:$IMAGE_TAG

      - name: Set up Kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/$DEPLOYMENT_NAME vue-container=$REGISTRY/${{ secrets.DOCKER_USERNAME }}/$IMAGE_NAME:$IMAGE_TAG --namespace $NAMESPACE
```

### ./k8s/deployment.yaml

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vue-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vue
  template:
    metadata:
      labels:
        app: vue
    spec:
      containers:
        - name: vue-container
          image: docker.io/${DOCKER_USERNAME}/vue-app:${IMAGE_TAG}
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: vue-service
  namespace: default
spec:
  type: LoadBalancer
  selector:
    app: vue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### service

```yml
apiVersion: v1
kind: Service
metadata:
  name: vue-service
spec:
  selector:
    app: vue-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

### Install kubectl

Intel Chip:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
```

### Validate the binary (optional):

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl.sha256"
```

### Validate the kubectl binary against the checksum file:

```bash
echo "$(cat kubectl.sha256)  kubectl" | shasum -a 256 --check
```

### If valid, the output is:

```bash
kubectl: OK
```

### Make the kubectl binary executable:

```bash
chmod +x ./kubectl
```

### Move the kubectl binary to a file location on your system PATH.

```bash
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

`Note`: Make sure /usr/local/bin is in your PATH environment variable.

### Test to ensure the version you installed is up-to-date:

```bash
kubectl version --client
```

Or use this for detailed view of version:

```bash
kubectl version --client --output=yaml
```

### After installing and validating kubectl, delete the checksum file:

```bash
rm kubectl.sha256
```

### To apply the config

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### KUBECONFIG GitHub Secret

To deploy from GitHub Actions:

1. Grab your ~/.kube/config file
1. Base64 encode it:

```bash
cat ~/.kube/config | base64
```

### How to Get DOCKER_USERNAME and DOCKER_PASSWORD

DOCKER_USERNAME

- Docker Hub Profile
- Or run:

  ```bash
  docker info | grep Username
  ```

DOCKER_PASSWORD

Create a Personal Access Token:

- Go to Docker Hub → Account Settings → Security
- Click "New Access Token"
- Name it (e.g., github-actions)
- Copy the generated token — you won't see it again

Then in GitHub:

- Go to your repo → Settings → Secrets and variables → Actions
- Click New repository secret
  - Name: DOCKER_USERNAME
  - Value: your Docker Hub username
- Click New repository secret
  - Name: DOCKER_PASSWORD
  - Value: the token from Docker
