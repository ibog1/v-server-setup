# v-server-setup

A complete, structured setup of a Linux V‑Server for secure SSH access and an NGINX website. This document follows the checklist: each step is placed in its own section, explicit testing is described.

## Table of contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [How to generate an SSH key](#how-to-generate-an-ssh-key)
- [Step 1: SSH hardening](#step-1-ssh-hardening)
- [Step 2: Install NGINX](#step-2-install-nginx)
- [Step 3: Replace default site](#step-3-replace-default-site)
- [Step 4: Configure Git](#step-4-configure-git)
- [Step 5: GitHub SSH and repository](#step-5-github-ssh-and-repository)
- [Testing](#testing)
- [Appendix](#appendix)

## Overview

This guide configures the server so SSH uses keys only, NGINX serves a custom index and 404 page, and Git/GitHub are set up over SSH for version control of documentation and safe configurations.

## Prerequisites

- Non‑root user with `sudo` on the server.
- Package manager access (`apt`) and a browser to verify the site.

## How to generate an SSH key

To securely access your server and use GitHub with SSH, you need an SSH key pair.  
If you do not already have one, generate it as follows:

> [!NOTE]  
> **Command (on your client/PC, not the server!):**.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
- When prompted, choose a file path or simply press Enter to accept the default.
- You can optionally set a secure passphrase.
- Your keys will be placed in `~/.ssh/id_ed25519` (private) and `~/.ssh/id_ed25519.pub` (public).

**Copy the public key to your server:**

ssh-copy-id username@your.server.address
or manually copy the contents of your `.pub` file into `~/.ssh/authorized_keys` on the server.

**Add the public key to GitHub:**

- Go to `Settings → SSH and GPG keys → New SSH key`, paste the contents of your public key file and save.

You are now ready to continue with the next steps!


## Step 1: SSH hardening

Purpose: Enforce key‑only SSH, block password logins, reduce attack surface.

**Procedure:**

- Copy your public key to the server user's `authorized_keys` if not present.
- Edit SSH daemon configuration file `/etc/ssh/sshd_config`:
    - `PubkeyAuthentication yes`
    - `PasswordAuthentication no`
    - `PermitRootLogin no` (recommended)
- Reload SSH to apply: `systemctl reload sshd`

> [!IMPORTANT]  
> Keep a second session open when changing SSH to avoid lockout.
> Test both negative and positive cases (see Testing).

## Step 2: Install NGINX

Purpose: Provide a web server to serve a custom landing page and error page.

**Procedure:**

- `apt update`
- `apt install -y nginx`
- Enable and start: `systemctl enable nginx` and `systemctl start nginx`

## Step 3: Replace default site

Purpose: Replace the default site with a custom index and 404 page.

**Procedure:**

- Place your custom `index.html` in `/var/www/html/index.html`.
- Create `/var/www/html/404.html` with a custom error message.
- Adjust `/etc/nginx/sites-available/default`:
    - `root /var/www/html;`
    - `index index.html;`
    - `error_page 404 /404.html;`
- Validate configuration: `nginx -t`
- Apply changes: `systemctl reload nginx`

**Verification:**

- Open `http://SERVER_IP` to see the new index page.
- Request a non-existing path to confirm the 404 page.

## Step 4: Configure Git

Purpose: Identify commits and set the default branch.

**Procedure:**

- Set user identity:
    - `git config --global user.name "YOUR NAME"`
    - `git config --global user.email "you@example.com"`
- Default branch:
    - `git config --global init.defaultBranch main`

## Step 5: GitHub SSH and repository

Purpose: Link local repository to GitHub via SSH.

**Procedure:**

- Ensure your SSH key is loaded (`ssh-agent` and `ssh-add`).
- Add your public key in GitHub under Settings → SSH and GPG keys.
- Verify connectivity (expect "authenticated, but no shell access"):
    - `ssh -T git@github.com`
- Create repository "v-server-setup" on GitHub (no README).
- In the local project folder:
    - `git init` (if not already initialized)
    - `git remote add origin git@github.com:YOUR_ACCOUNT/v-server-setup.git`
    - `git add .`
    - `git commit -m "Initial documentation"`
    - `git push -u origin main`

## Testing

Perform these checks before submission:

- **SSH authentication:**  
  - Negative (should fail):  
    `ssh -o PubkeyAuthentication=no user@SERVER`
  - Positive (should succeed):  
    `ssh user@SERVER`
- **NGINX:**  
  - `nginx -t` returns "syntax is ok; test is successful"
  - `systemctl reload nginx` applies config without errors
  - Browser shows custom index page; non-existing paths get the custom 404
- **GitHub & push:**  
  - `ssh -T git@github.com` shows successful authentication message
  - `git push -u origin main` completes, README appears on GitHub

## Appendix

**Common paths:**

- SSH config: `/etc/ssh/sshd_config`
- NGINX config: `/etc/nginx/nginx.conf` & `/etc/nginx/sites-available/default`
- Web root: `/var/www/html`

**Service commands:**

- `systemctl status|reload|restart sshd`
- `systemctl status|reload|restart nginx`

---
