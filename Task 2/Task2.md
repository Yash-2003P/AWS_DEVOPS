# Task 2 — Linux Basics: Nginx Install, Status, Restart, Monitoring (20 Marks)

## Solution — What I Did Step by Step

### Step 1 — Confirm OS Before Doing Anything

```bash
cat /etc/redhat-release
```
- Confirms I am actually on RHEL before running any RHEL-specific commands. I already checked `/etc/os-release` in Task 1 from WSL2 side, this confirms the same from inside the EC2 instance.

### Step 2 — Update Packages and Install Nginx

```bash
sudo dnf update -y
```
- Updates all installed packages to latest versions. The `-y` flag auto-confirms so it does not stop and ask for confirmation. Used `dnf` (RHEL's package manager), not `yum` or `rpm`.

```bash
sudo dnf install nginx -y
```
- Installs the Nginx web server. `-y` to auto-confirm.

```bash
sudo dnf list installed nginx
```
- Confirms nginx is actually installed and shows its version.

### Step 3 — Configure OS-Level Firewall (firewalld)

This is a separate thing from the AWS Security Group. The Security Group controls traffic entering AWS network. `firewalld` controls traffic entering the OS itself. Both need port 80 open, or the website will not load from outside.

I had studied firewalld before in RHCSA preparation, so I also checked Red Hat's own docs to confirm the correct way to allow traffic for a specific service (like http) using `firewall-cmd`.

```bash
sudo dnf install firewalld -y
```
- Installs firewalld (it may or may not come pre-installed depending on the AMI).

```bash
sudo systemctl status firewalld
```
- Checks if firewalld is running. If it shows inactive or not found, you need to install and start it.

```bash
sudo systemctl enable --now firewalld
```
- `enable` means it starts on every boot. `--now` also starts it right now. One command does both.

```bash
sudo firewall-cmd --list-all
```
- Shows all current firewall rules and allowed services/ports. Before making changes, I checked what was already open.

```bash
sudo firewall-cmd --permanent --add-service=http
```
- Permanently opens the HTTP service (port 80) in firewalld. `--permanent` means the rule survives reboot. Without `--permanent`, the rule is lost on reboot.

```bash
sudo firewall-cmd --permanent --add-port=80/tcp
```
- Explicitly opens TCP port 80 as well. This is another way to do the same thing as `--add-service=http`. I added both to be sure.

```bash
sudo firewall-cmd --reload
```
- Reloads firewalld so the `--permanent` rules take effect immediately. Without this, permanent rules only apply after a reboot.

```bash
sudo firewall-cmd --list-all
```
- Second check to confirm `http` now shows up under `services` in the output.

### Step 4 — Quick SELinux Check

```bash
getenforce
```
- Shows the current SELinux mode. Output was `Enforcing`. This is the correct default for RHEL — I did not change it. SELinux controls what files a process is allowed to read. It is a different layer from both the AWS Security Group and firewalld. It became important later in Task 3.

### Step 5 — Enable and Start Nginx

```bash
sudo systemctl status nginx
```
- Checks nginx status before starting. Should show `inactive (dead)` since we just installed it.

```bash
sudo systemctl enable --now nginx
```
- Enables nginx to start on boot AND starts it right now in one command.

```bash
sudo systemctl status nginx
```
- Confirms nginx is now `active (running)`. Look for the green `active (running)` text in the output.

### Step 6 — Practice Service Control (restart / stop / start)

```bash
sudo systemctl restart nginx
```
- Restarts nginx. Used to apply config changes.

```bash
sudo systemctl stop nginx
```
- Stops nginx completely. I did this to see what `inactive (dead)` looks like in status output.

```bash
sudo systemctl status nginx
```
- Confirmed it shows `inactive (dead)`.

```bash
sudo systemctl restart nginx
```
- Started it again.

```bash
sudo systemctl status nginx
```
- Confirmed `active (running)` again.

### Step 7 — Monitoring Commands (Disk, Memory, Process)

```bash
df -h
```
- Shows disk usage in human-readable format. `-h` converts bytes to KB/MB/GB.

```bash
free -h
```
- Shows memory (RAM) usage in human-readable format.

```bash
ps aux | grep nginx
```
- Shows all running processes that have "nginx" in their name. Confirms nginx master and worker processes are running.

```bash
top
```
- Shows real-time system resource usage (CPU, memory, processes). Press `q` to exit.

**Additional commands I used:**

```bash
sudo ss -tulpn | grep :80
```
- Confirms something is actually listening on port 80 and shows which process (should show nginx).

```bash
free -h | awk 'NR==2{print $3, $2}'
```
- Pulls just the used and total memory numbers from the `free -h` output. A simple use of `awk` to filter out just what I need.

```bash
du -sh /var/log/* | sort -rh | head -5
```
- Shows the 5 biggest log directories, largest first. Useful to know what is taking disk space.

**Screenshot taken:** `sudo systemctl status nginx` showing `active (running)`.

## Time Taken

**1:53 PM – 2:27 PM (~34 minutes)**

This includes the actual command execution time plus reading Nginx official documentation and Red Hat firewalld documentation to get the port-opening steps right. I did not just copy-paste commands — I read the docs to understand why each flag is used.

## Problems I Faced

No real problem in this task. The install and service start worked fine on the first try. The only thing I had to be careful about was making sure both the AWS Security Group (Task 1) and the OS-level firewalld (this task) both had port 80 open — opening one does not automatically open the other.

## What I Learned

- RHEL package manager is `dnf`, not `apt` (which is what Ubuntu uses).
- AWS Security Group (AWS side) and firewalld (OS side) are two separate walls. Opening one does not open the other. Both need port 80 for the website to be reachable from the internet.
- SELinux is on by default in RHEL. Most Ubuntu-based tutorials do not mention it at all because Ubuntu does not use SELinux.
- Difference between `systemctl status`, `restart`, `stop`, and `enable --now`.
- Basic Linux monitoring commands: `df -h`, `free -h`, `ps aux`, `top`, `ss -tulpn`.

## References

- Red Hat documentation — Deploying web servers and reverse proxies (Nginx setup on RHEL): https://docs.redhat.com/en//red_hat_enterprise_linux/10/html/deploying_web_servers_and_reverse_proxies/setting-up-and-configuring-nginx
