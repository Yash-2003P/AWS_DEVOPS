# Task 3 — Host a Simple Website (20 Marks)
# Create a basic HTML page containing your Name, College, Branch, Email, and Current Date. Replace the default Nginx page with it.
# Deliverables: Website accessible via EC2 Public IP and browser screenshot.

## Solution — What I Did Step by Step

### Step 1 — Check Default Nginx Page Is Working First

```bash
curl http://127.0.0.1:80
```
- Tests if nginx is serving anything at all on the local machine. This confirms nginx itself is working, separate from any network or firewall question.

```bash
cat /usr/share/nginx/html/index.html | head -n 10
```
- Shows the first 10 lines of the default nginx page. This is the stock "Welcome to nginx" page that comes pre-installed.

```bash
sudo nginx -t
```
- Tests the nginx configuration file for syntax errors. Should output `syntax is ok` and `test is successful`.

```bash
curl http://<ec_public_ip>
```
- Tests if the default page is also reachable from outside using the public IP.

```bash
curl http://<ec_public_ip>:80 | grep "If you are the website"
```
- Searched for specific text in the default page to confirm it is the nginx welcome page, not an error.

```bash
curl http://<ec_public_ip>:80 | tail -10
```
- Shows the last 10 lines of the response to further verify the page content.

### Step 2 — Tried Fetching HTML from GitHub (Did Not Use This Approach)

First I tried downloading the HTML file from my GitHub repo directly onto the server:

```bash
curl -o index.html https://raw.githubusercontent.com/<user>/<repo>/main/index.html
```
- `-o index.html` saves the downloaded content to a file named `index.html`.

```bash
wget -O index.html https://raw.githubusercontent.com/<user>/<repo>/main/index.html
```
- `-O` (capital O) for wget does the same thing — saves with the exact filename.

**Important note about GitHub links:** The `github.com/<user>/<repo>/blob/main/file` link shows GitHub's viewer page (full of GitHub's own HTML/JavaScript). To get the actual raw file, you must use `raw.githubusercontent.com` and drop `/blob` from the path.

This approach technically worked but was slow and fiddly over SSH. So I dropped it and wrote the file directly on the server using `vi` instead — faster and more reliable for a small one-page file.

### Step 3 — Created HTML Page Using vi Editor

```bash
cd ~
```
- Went to my home directory.

```bash
vi index.html
```
- Opened the `vi` text editor to create a new file called `index.html`.

**How to use vi (step by step):**

1. Run `vi index.html` — opens a blank file.
2. Press `i` — enters Insert mode (now you can type/paste).
3. Paste the HTML code (or type it manually).
4. Press `Esc` — exits Insert mode.
5. Type `:wq!` and press Enter — force-saves and quits.

**HTML boilerplate I pasted** (I will replace the placeholder text with my actual details):

```html
<!DOCTYPE html>
<html>
<head><title>DevOps Intern Assignment</title></head>
<body>
  <h1>Your Name</h1>
  <p>College: Your College Name</p>
  <p>Branch: Your Branch</p>
  <p>Email: your.email@example.com</p>
  <p>Date: <!-- fill in current date --></p>
</body>
</html>
```

### Step 4 — Replaced Default Nginx Page With My Page

```bash
sudo cp ~/index.html /usr/share/nginx/html/index.html
```
- Copies my HTML file into nginx's web root directory. I used `cp`, not `mv`, on purpose (see Problem 1 below for why).

```bash
sudo restorecon -v /usr/share/nginx/html/index.html
```
- Fixes the SELinux file label. On RHEL, nginx is only allowed to read files with the `httpd_sys_content_t` label. Ran this as a safety step even after `cp` (which usually sets the label correctly on its own).

```bash
cat /usr/share/nginx/html/index.html | head -n 10
```
- Verified the file was copied correctly by checking the first 10 lines.

### Step 5 — Verified and Reloaded Nginx

```bash
sudo nginx -t
```
- Tested nginx config for syntax errors before reloading.

```bash
sudo systemctl reload nginx
```
- Reloads nginx so it picks up the new file. `reload` is smoother than `restart` — it applies changes without dropping active connections.

```bash
curl http://localhost
```
- Checked if the new page is served from the server side. I did this before opening the browser so I could tell if any problem was server-side or browser-side.

```bash
curl http://<ec_public_ip> | tail -n 10
```
- Confirmed the new page is reachable from outside via the public IP.

```bash
sudo systemctl status nginx
```
- Final check that nginx is still `active (running)`.

**Screenshot taken:** Website loading in browser at `http://<ec_public_ip>` showing my name, college, branch, email, and date.

Later, after the Elastic IP bonus task, I also checked the site at `http://<ec2_elastic_public_ip>` to confirm it works on the new fixed IP.

## Time Taken

**2:39 PM – 2:59 PM (~20 minutes)**

There was also a block from **2:29 PM – 2:39 PM (~10 minutes)** right before this where I was reading Nginx docs and testing `wget`/`curl` to fetch the HTML file from GitHub. That approach was slow so I switched to using `vi` directly.

## Problems I Faced

**1. 403 Forbidden error after replacing the page.**

While I was testing (created `index.html`, deleted it, made it again a few times with `vi`), on one attempt I moved the file with `mv` instead of `cp`. RHEL's SELinux blocked nginx from reading it.

- **How I found the cause:**
  ```bash
  ls -Z /usr/share/nginx/html/index.html
  ```
  - Shows the SELinux label on the file. It showed `user_home_t` instead of `httpd_sys_content_t`.
  ```bash
  sudo ausearch -m avc -ts recent | grep nginx
  ```
  - Shows the actual kernel-level SELinux denial log. Confirmed nginx was denied access.

- **Fix:**
  ```bash
  sudo restorecon -Rv /usr/share/nginx/html/
  ```
  - Restores the correct SELinux labels recursively.
  ```bash
  sudo nginx -t
  sudo systemctl reload nginx
  curl http://localhost
  ```
  - After `restorecon`, re-tested config, reloaded nginx, and confirmed `curl` returns the page with `200 OK`.

**2. Browser still showing old default page after reload.**

This was not a server problem. The browser was showing a cached copy of the old page. I confirmed the new content was actually live by running `curl http://localhost` directly on the server (which bypasses browser cache completely). Then did a hard refresh in the browser (`Ctrl+Shift+R`) and the new page appeared.

## Where I Searched For the Fix

- **Red Hat documentation** — Deploying web servers and reverse proxies (Nginx setup on RHEL): https://docs.redhat.com/en//red_hat_enterprise_linux/10/html/deploying_web_servers_and_reverse_proxies/setting-up-and-configuring-nginx
- **General web search** — Searched for "403 Forbidden Nginx SELinux RHEL" which confirmed `restorecon` is the standard fix. This SELinux behaviour is not covered in most Ubuntu-based Nginx guides.

## What I Learned

- Always use `cp`, not `mv`, when putting content into `/usr/share/nginx/html/` on RHEL. `cp` creates a new file and RHEL labels it correctly. `mv` keeps the old label from wherever the file was before.
- `restorecon` fixes the SELinux label and is safe to run even if you are not sure it is needed.
- `curl` on the server itself is the fastest way to check if an issue is server-side or just browser cache.
- The correct URL to download a raw file from GitHub is `raw.githubusercontent.com`, not `github.com/.../blob/...`.
