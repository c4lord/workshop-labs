# Exercise 4: Advanced Patterns

**Objective:** Master advanced GitHub Actions features including secrets, environments, conditionals, and concurrency.

**Duration:** 30 minutes

---

## Overview

In this exercise, you'll:
- Use secrets for sensitive data
- Set up protected environments
- Add conditional logic
- Implement concurrency controls
- Learn about reusable workflows

---

## Step 1: Secrets

Store sensitive data securely:

**1. Add a secret:**
- Go to your repository on GitHub
- Settings → Secrets and variables → Actions
- Click "New repository secret"
- Name: `NPM_TOKEN`
- Value: Your npm token

**2. Use in workflow:**

```yaml
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: npm ci

      - name: Publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## Step 2: Environments

Create protected environments for deployments:

**1. Create an environment:**
- Go to your repository
- Settings → Environments
- Click "New environment"
- Name: `production`
- Configure protection rules

**2. Add to workflow:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: echo "Deploying to production"
```

**Environment protections:**
- Required reviewers
- Wait timer
- Deployment branches

---

## Step 3: Conditionals

Run steps conditionally:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Only run on pull requests
      - name: PR comment
        if: github.event_name == 'pull_request'
        run: echo "This is a PR"

      # Skip on specific paths
      - name: Build
        if: "!contains(github.event.head_commit.message, '[skip ci]')"
        run: npm run build

      # Conditional matrix
      - name: Build Windows
        if: matrix.os == 'windows-latest'
        run: echo "Building Windows"
```

**Common conditions:**
```yaml
if: github.event_name == 'pull_request'
if: github.event_name == 'schedule'
if: github.ref == 'refs/heads/main'
if: contains(github.event.head_commit.message, 'deploy')
if: always() || failure()
```

---

## Step 4: Concurrency

Prevent duplicate runs:

```yaml
name: CI

on:
  push:
    branches: [main]

# Cancel in-progress runs for the same branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

**Use cases:**
- Cancel old PR builds when new commits push
- Prevent duplicate scheduled runs
- Single build per branch

---

## Step 5: Reusable Workflows

Create reusable workflow templates:

**`.github/workflows/reusable-test.yml`:**
```yaml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: '20'
    secrets:
      npm_token:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}

      - name: Install
        run: npm ci

      - name: Test
        run: npm test
```

**Use in another workflow:**

```yaml
jobs:
  test-node-18:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '18'

  test-node-20:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
```

---

## Step 6: Paths Filter

Only run workflow on specific file changes:

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'package.json'
      - 'package-lock.json'
  pull_request:
    paths:
      - 'src/**'
      - 'package.json'
```

---

## Complete Advanced Workflow

```yaml
name: Advanced CI/CD

on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  NODE_VERSION: '20'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
```

---

## Verification Checklist

- [ ] Secrets configured and used
- [ ] Environments set up with protection
- [ ] Conditionals work correctly
- [ ] Concurrency cancels duplicate runs
- [ ] Reusable workflow created

---

## What You've Learned

✅ Using secrets securely  
✅ Protected environments  
✅ Conditional execution  
✅ Concurrency controls  
✅ Reusable workflows  
✅ Path filtering  

---

## Next Steps

Move on to [Exercise 5: Security Hardening](../exercise-05-security/)
