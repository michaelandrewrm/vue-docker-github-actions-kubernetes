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

```

# File: .github/workflows/hello-world.yml

name: Hello World Workflow

# When anyone pushes code it will run a job on a fresh Ubuntu server that echoes a message.
on: [push]

jobs:
say_hello:
runs-on: ubuntu-latest
steps: - name: Print a message
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

```
on:
  push:
    branches:
      - main
```

Manual trigger (very important for controlled deployments):

```
on:
  workflow_dispatch:
```

Cron schedule (run every day at midnight UTC):
Cron syntax: `min` `hour` `day` `month` `weekday`

```
on:
  schedule:
    - cron: '0 0 * * *'
```

### 🔥 Part 2: Jobs (aka "How you organize the work")

A job is a collection of steps that run in the same environment (called a runner).

Basic job structure:

```
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

```
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

```
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
