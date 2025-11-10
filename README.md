# v-server-setup

A complete, structured setup of a Linux V‑Server for secure SSH access and an NGINX website, documented in Markdown with a clickable table of contents. This document follows the checklist: each step is in its own section, testing is explicit, and no secrets are committed.
Contents
•	Overview
•	Prerequisites
•	Step 1: SSH hardening
•	Step 2: Install NGINX
•	Step 3: Replace default site
•	Step 4: Configure Git
•	Step 5: GitHub SSH and repository
•	Testing
•	Security and privacy
•	Appendix
Overview
This guide configures the server so SSH uses keys only, NGINX serves a custom index and 404 page, and Git/GitHub are set up over SSH for version control of documentation and safe config where appropriate.
Prerequisites
•	Non‑root user with sudo on the server.
•	SSH key pair; public key available to add to authorized_keys and to GitHub.
•	Package manager access (apt) and a browser to verify the site.
Step 1: SSH hardening
Purpose: enforce key‑only SSH, block password logins, reduce attack surface.
Procedure:
Copy your public key to the server user's authorized_keys if not present.
• Edit SSH daemon configuration file /etc/ssh/ sshd_config:
• PubkeyAuthentication yes
• PasswordAuthentication no
• PermitRootLogin no (recommended)
• Reload SSH to apply:
• systemct reload sshd
Notes:
• Keep a second session open when changing
SSH to avoid lockout.
• Test both negative and positive cases (see
Testing).
Step 2: Install NGINX
Purpose: provide a web server to serve a custom landing page and error page.
Procedure:
• apt update
⁠• apt install -y nginx
• Enable and start:
• systemctl enable nginx
• systemctl start nginx
Step 3: Replace default site
Purpose: display a custom index and a custom 404 page.
Procedure:
• Place your index.html in /var/www/html/ index.html.
• In /etc/nginx/sites-available/default, ensure:
• root /var/www/html;
• index index.html;
• error_page 404 /404.html;
• Add /var/www/html/404.html with a simple message.
• ⁠Validate and apply:
• nginx -t
• systemctl reload nginx
Verification:
Open http://SERVER_IP to see your index.
• Request a non-existing path to confirm the 404 page.
Step 4: Configure Git
Purpose: identify commits and set main as default branch.
Procedure:
• git config -global user.name "YOUR NAME"
• git config -global user.email"you@example.com"
* git config -global init.defaultBranch main
Step 5: GitHub SSH and repository
Purpose: version control and remote backup of documentation.
Procedure:
• Ensure your SSH key is loaded (ssh-agent + ssh-add).
• Add the public key to GitHub under Settings →
SSH and GPG keys.
• Verify connectivity (no shell access expected):
• ssh-T -Tgit@github.com
• Create the empty repository v-server-setup on
GitHub (no README).
• In your local folder:
• git init (already done)
• git remote add origin
git@github.com:YOUR_ACCOUNT/v-
server-setup.git
git add.
• git commit -m "Initial documentation"
• git push -u origin main
Testing
Perform these checks before submission; record the outcomes if desired:
• SSH key-only authentication:
Negative (should fail): ssh -o
PubkeyAuthentication=no user@SERVER
• Positive (should succeed): ssh user@SERVER
• NGINX configuration and availability:
• nginx-t returns syntax is ok; test is successful
• systemctl reload nginx applies changes without errors
• Browser shows custom index and 404 page
• GitHub connectivity and push:
• ssh -T git@github.com shows successful authentication message
• git push -u origin main completes and
README appears on GitHub
Security and privacy
Exclude sensitive files with gitignore; prefer environment variables or a dedicated secret store for credentials.
• Never add content of ~/.ssh or confidential files from /etc to version control.
• Always review diffs before pushing to ensure no confidential data is included.
Appendix
Common paths:
• SSH: /etc/ssh/sshd_config
• NGINX: /etc/nginx/nginx.conf, /etc/nginx/
sites-available/default
• Web root: /var/www/html
Service helpers:
• systemctl status|reload|restart sshd
• systemctl status|reload|restart nginx
