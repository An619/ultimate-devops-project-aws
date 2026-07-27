This is an **excellent CI pipeline** because it demonstrates how a real company automates code validation, Docker image creation, and Kubernetes deployment updates.

Instead of explaining it line by line, let's first understand **the story** of what happens.

---

# The Story

Suppose you're working at Amazon.

You fix a bug in the Product Catalog service.

You push your code to GitHub.

```text
Developer
      │
      ▼
Git Push
```

Now GitHub says

> "Someone changed the code."

It immediately starts this workflow.

Everything written in this YAML is simply answering one question:

> **"What should GitHub do after code changes?"**

---

# Overall Pipeline

The pipeline looks like this:

```text
Pull Request
      │
      ▼
Build
      │
      ▼
Code Quality
      │
      ▼
Docker Build & Push
      │
      ▼
Update Kubernetes YAML
```

Now let's understand every part.

---

# Part 1

```yaml
name: product-catalog-ci
```

This is simply the workflow name.

GitHub Actions UI will show

```text
product-catalog-ci
```

Think of it as naming your Jenkins job.

---

# Part 2

```yaml
on:
  pull_request:
    branches:
      - main
```

This is the trigger.

Meaning

> **When should this workflow start?**

Answer

Whenever someone creates or updates a Pull Request targeting the `main` branch.

Example

```text
feature-login

↓

Pull Request

↓

main
```

GitHub automatically starts the pipeline.

Without this section, GitHub wouldn't know **when** to run it.

---

# Part 3

```yaml
jobs:
```

A workflow consists of one or more jobs.

Think

```text
Workflow

↓

Job 1

↓

Job 2

↓

Job 3
```

In your case

```text
Workflow

↓

Build

↓

Code Quality

↓

Docker

↓

Update K8s
```

---

# Job 1

```yaml
build:
```

This job checks

> "Can the application compile?"

---

## runs-on

```yaml
runs-on: ubuntu-latest
```

GitHub now creates a temporary Ubuntu virtual machine.

Think

```text
GitHub

↓

Creates fresh Ubuntu VM

↓

Runs pipeline

↓

Deletes VM
```

Every workflow gets a clean machine.

Nothing remains from previous runs.

---

# Step 1

```yaml
- name: checkout code
```

```yaml
uses: actions/checkout@v4
```

Remember

GitHub Actions VM is empty.

```text
Ubuntu VM

↓

No source code
```

This step runs

```bash
git clone
```

internally.

Now

```text
Ubuntu VM

↓

Source Code
```

exists.

Without checkout

there is nothing to build.

---

# Step 2

```yaml
Setup Go 1.22
```

GitHub VM doesn't necessarily have Go installed.

So this action installs Go.

Equivalent to

```bash
sudo apt install golang
```

(Conceptually.)

Now Go compiler exists.

---

# Step 3

```yaml
cd src/product-catalog
```

Move into

```text
src/product-catalog
```

because that's where

```text
main.go
```

exists.

---

# Step 4

```bash
go mod download
```

Remember

Go has dependencies.

Like Java has

```xml
pom.xml
```

Node has

```text
package.json
```

Go has

```text
go.mod
```

This command downloads all required libraries.

Example

```text
GitHub

↓

Download Gin

↓

Download OpenTelemetry SDK

↓

Download Logging Library
```

Now project is ready.

---

# Step 5

```bash
go build
```

Compile.

```text
Source Code

↓

Machine Code

↓

Executable
```

If compilation fails

Pipeline stops.

---

# Step 6

```bash
go test ./...
```

Runs every unit test.

Suppose

```text
Login Test

Payment Test

Cart Test
```

all pass.

Pipeline continues.

If

```text
1 Test Failed
```

Pipeline stops.

---

# Second Job

```yaml
code-quality:
```

Notice

This isn't building.

It's checking code quality.

---

Again

Checkout

↓

Install Go

↓

Run

```yaml
golangci-lint
```

---

What is golangci-lint?

Imagine

You wrote

```go
x := 10
```

Never used.

Compiler may allow it.

Linter says

```text
Unused Variable
```

Another example

```go
if(a==true)
```

Linter says

Bad coding style.

Another

```go
fmt.Println(password)
```

Possible security issue.

So

```text
Build

↓

Checks

Can code compile?

Linter

↓

Checks

Is code well written?
```

Different things.

---

# Third Job

```yaml
docker:
```

Now code is good.

Let's package it.

Notice

```yaml
needs: build
```

Meaning

Don't start

until Build succeeds.

Dependency

```text
Build

↓

Docker
```

If Build fails

Docker never runs.

---

# Login

```yaml
docker/login-action
```

Pipeline logs into Docker Hub.

Using

```yaml
${{ secrets.DOCKER_USERNAME }}
```

GitHub Secrets.

Password never appears in YAML.

Very important.

---

# Docker Build

```yaml
docker/build-push-action
```

Equivalent to

```bash
docker build
```

and

```bash
docker push
```

together.

---

Context

```yaml
context:
```

Where source code exists.

Dockerfile

```yaml
file:
```

Which Dockerfile to use.

---

Push

```yaml
push: true
```

Meaning

Don't just build.

Upload.

---

Tag

```yaml
tags:

username/product-catalog:${{ github.run_id }}
```

Suppose

Run ID

```text
155
```

Image becomes

```text
product-catalog:155
```

Next run

```text
156
```

Next image

```text
product-catalog:156
```

Every pipeline produces a unique image.

Very useful.

---

# Fourth Job

```yaml
updatek8s:
```

This is where GitOps begins.

Notice

```yaml
needs: docker
```

Meaning

Don't update Kubernetes

unless Docker image exists.

---

Checkout

Again download repository.

---

Now interesting part.

```bash
sed -i
```

What is

```bash
sed
```

Think

Search

↓

Replace

Suppose deployment.yaml currently has

```yaml
image:

product-catalog:120
```

Pipeline replaces it with

```yaml
image:

product-catalog:155
```

Automatically.

---

Then

```bash
git add
```

Stages changes.

---

```bash
git commit
```

Creates commit.

---

```bash
git push
```

Pushes updated Kubernetes YAML.

Now GitHub contains

```yaml
image:

product-catalog:155
```

---

# Where is Kubernetes?

Notice

This pipeline

**never runs**

```bash
kubectl apply
```

Many beginners miss this.

Instead

Pipeline only changes Git.

Then

```text
GitHub

↓

ArgoCD notices

↓

kubectl apply

↓

Kubernetes Updated
```

This is called

GitOps.

---

# Why not directly deploy?

Old way

```text
CI

↓

SSH

↓

kubectl apply
```

Problem

Git no longer reflects reality.

---

Modern

```text
CI

↓

Update Git

↓

ArgoCD

↓

Cluster
```

Git always matches production.

Much safer.

---

# Complete Pipeline

```text
Developer creates Pull Request
            │
            ▼
GitHub Actions Triggered
            │
            ▼
──────────────────────────────────────
Job 1 : Build
──────────────────────────────────────
Checkout Code
        │
        ▼
Install Go
        │
        ▼
Download Dependencies
        │
        ▼
Compile
        │
        ▼
Run Unit Tests
        │
        ▼
Success
            │
            ▼
──────────────────────────────────────
Job 2 : Code Quality
──────────────────────────────────────
Checkout Code
        │
        ▼
Install Go
        │
        ▼
Run golangci-lint
        │
        ▼
Success
            │
            ▼
──────────────────────────────────────
Job 3 : Docker
──────────────────────────────────────
Checkout Code
        │
        ▼
Login to Docker Hub
        │
        ▼
Build Docker Image
        │
        ▼
Push Image
(product-catalog:155)
            │
            ▼
──────────────────────────────────────
Job 4 : Update Kubernetes
──────────────────────────────────────
Checkout Repository
        │
        ▼
Replace Old Image Tag
        │
        ▼
Commit
        │
        ▼
Push Updated Manifest
            │
            ▼
GitHub Repository Updated
            │
            ▼
ArgoCD Detects Change
            │
            ▼
kubectl apply
            │
            ▼
Deployment Updated
            │
            ▼
Kubernetes pulls new Docker Image
            │
            ▼
Rolling Update Starts
            │
            ▼
Users get the new version
```

---

# One issue I noticed in this pipeline

There's a subtle but important point.

Your workflow has **four jobs**:

* `build`
* `code-quality`
* `docker`
* `updatek8s`

The `docker` job has:

```yaml
needs: build
```

but **it does not depend on `code-quality`**.

That means the execution looks like this:

```text
           build
          /     \
         ▼       ▼
code-quality   docker
                  │
                  ▼
             updatek8s
```

So if the linter (`code-quality`) fails **after** `docker` has already started, it's possible for a Docker image to be built and pushed even though the code quality checks failed.

A safer dependency graph is:

```yaml
docker:
  needs:
    - build
    - code-quality
```

which gives you:

```text
        build
          │
          ▼
    code-quality
          │
          ▼
       docker
          │
          ▼
     updatek8s
```

This ensures that **only code that compiles, passes tests, and passes quality checks is packaged into a Docker image and prepared for deployment.** This is the pattern you'll commonly see in production CI pipelines.
