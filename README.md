# AWS DevOps Engineer Intern Assignment (1 Day)

## Objective
Deploy a simple website on an AWS EC2 instance and document the process.

---

## IAM Setup

All EC2 work was performed using a dedicated IAM user, not the root account.

- Logged into the AWS root account.
- Created an IAM group named `aws_devops_group`.
- Created an IAM user named `aws_devops_intern`.
- Added the user to the group.
- Attached the **`AdministratorAccess`** managed policy to the group (for a real production task, **`AmazonEC2FullAccess`** is the safer, correct policy since the assignment only needs EC2 access).
- Logged out of root and logged back in as `aws_devops_intern` for all further work.

---

## Task 1: Create an AWS EC2 Instance

- Navigated to **EC2 Console** → **Instances** → **Launch Instance**.
- **Name:** `RHEL10`
- **AMI:** Red Hat Enterprise Linux 10 (official Red Hat AMI from AWS Marketplace).
- **Instance type:** `t2.micro` (free-tier eligible).
- **Key pair:** Created a new `.pem` key pair and downloaded the private key.
- **Network settings:** Default VPC, default subnet, auto-assign public IP enabled.
- **Storage:** Default EBS volume (gp3, 8 GB).

**Security Group inbound rules:**
- **SSH (22)** — Source: My IP only.
- **HTTP (80)** — Source: Anywhere-IPv4 (`0.0.0.0/0`).
- **HTTPS (443)** — not opened (no SSL/TLS certificate configured).

Waited for **2/2 status checks passed** and confirmed the instance state as `running` before proceeding.

**Connecting via SSH (from WSL2):**

```bash
cat /etc/os-release
```

```bash
sudo mv /mnt/c/Users/vish9/Downloads/rhel10_keys.pem /home/buntu1/
```

```bash
ls -l rhel10_keys.pem
```

```bash
chmod 400 rhel10_keys.pem
```

```bash
ls -l rhel10_keys.pem
```

```bash
ssh -i rhel10_keys.pem ec2-user@<ec_public_ip>
```

**Note:** AWS assigns a dynamic public IP by default, and it changes every time the instance is stopped and started. The Elastic IP allocated in the Bonus task fixes this by giving the instance a static public IP.

---

## Task 2: Linux Basics — Nginx, Firewall, Monitoring

```bash
cat /etc/redhat-release
```

```bash
sudo dnf update -y
```

```bash
sudo dnf install nginx -y
```

```bash
sudo dnf list installed nginx
```

**firewalld (OS-level firewall, separate from the AWS Security Group — both must allow port 80):**

```bash
sudo dnf install firewalld -y
```

```bash
sudo systemctl status firewalld
```

```bash
sudo systemctl enable --now firewalld
```

```bash
sudo firewall-cmd --list-all
```

```bash
sudo firewall-cmd --permanent --add-service=http
```

```bash
sudo firewall-cmd --permanent --add-port=80/tcp
```

```bash
sudo firewall-cmd --reload
```

```bash
sudo firewall-cmd --list-all
```

**SELinux check:**

```bash
getenforce
```

**Enabling Nginx:**

```bash
sudo systemctl status nginx
```

```bash
sudo systemctl enable --now nginx
```

```bash
sudo systemctl status nginx
```

**Service control practice:**

```bash
sudo systemctl restart nginx
sudo systemctl stop nginx
sudo systemctl status nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

**Monitoring commands:**

```bash
df -h
free -h
ps aux | grep nginx
top
sudo ss -tulpn | grep :80
free -h | awk 'NR==2{print $3, $2}'
du -sh /var/log/* | sort -rh | head -5
```

---

## Task 3: Host a Simple Website

**Verifying the default page:**

```bash
curl http://127.0.0.1:80
```

```bash
cat /usr/share/nginx/html/index.html | head -n 10
```

```bash
sudo nginx -t
```

```bash
curl http://<ec_public_ip>
```

```bash
curl http://<ec_public_ip>:80 | grep "If you are the website"
```

```bash
curl http://<ec_public_ip>:80 | tail -10
```

**Attempt to fetch HTML from GitHub (not used in final approach):**

```bash
curl -o index.html https://raw.githubusercontent.com/<user>/<repo>/main/index.html
```

```bash
wget -O index.html https://raw.githubusercontent.com/<user>/<repo>/main/index.html
```

This was slow and cumbersome over SSH, so it was abandoned in favor of creating the file directly on the server with `vi`.

**Creating the custom HTML page with `vi`:**

```bash
cd ~
vi index.html
```

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

**Replacing the default Nginx page** — `cp` was used instead of `mv` so the new file inherits the correct SELinux context (`httpd_sys_content_t`) from the destination directory:

```bash
sudo cp ~/index.html /usr/share/nginx/html/index.html
```

```bash
cat /usr/share/nginx/html/index.html | head -n 10
```

```bash
sudo nginx -t
```

```bash
sudo systemctl reload nginx
```

```bash
curl http://localhost
```

```bash
curl http://<ec_public_ip> | tail -n 10
```

```bash
sudo systemctl status nginx
```

---

## Task 4: Git and GitHub

Files were uploaded through the GitHub website UI, organised into folders by task.

**Equivalent git CLI workflow:**

```bash
cd ~/my-project
git init
git branch -M main
git add .
git status
git commit -m "Initial commit: AWS DevOps assignment — EC2 setup, Nginx, website hosting"
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

**Note on GitHub links:**
- Viewer page — `github.com/<user>/<repo>/blob/main/file.html` — opens GitHub's wrapped web viewer.
- Raw file — `raw.githubusercontent.com/<user>/<repo>/main/file.html` — serves the actual raw content, needed for `curl`/`wget`.

---

## Task 5: Documentation

A consolidated PDF/DOCX report was prepared covering the objective, AWS services used, IAM setup, all Linux commands with explanations, problems encountered and their fixes, key learnings, and reference links.

---

## Bonus: Attach an Elastic IP

- Navigated to **EC2 Console** → **Network & Security** → **Elastic IPs**.
- Clicked **"Allocate Elastic IP address"**, left the network scope as default, and clicked **Allocate**.
- Selected the new Elastic IP → **Actions** → **"Associate Elastic IP address"**, chose the `RHEL10` instance, and clicked **Associate**.

```bash
ssh -i rhel10_keys.pem ec2-user@<ec2_elastic_public_ip>
```

```bash
curl http://<ec2_elastic_public_ip>
```

Nginx needed a restart to rebind to the new network interface:

```bash
sudo systemctl restart nginx
```

```bash
curl http://<ec2_elastic_public_ip>
```

```bash
scp -i rhel10_keys.pem ec2-user@<ec2_elastic_public_ip>:/home/ec2-user/abc.txt /mnt/c
```

---

## Repository Structure

```
.
├── BONUS/
│   └── Bonus.md
├── Task 1/
│   ├── Task1.md
│   └── ec2.webp
├── Task 2/
│   ├── 21.png
│   └── Task2.md
├── Task 3/
│   ├── Task3.md
│   └── index.html
├── Task 4/
│   └── Task4.md
├── Task 5/
│   ├── Task5.md
│   └── Documentation.pdf
└── README.md
```

---

## Live Website

`http://<ec2_elastic_public_ip>`

## Notes
 
This setup covers a single EC2 instance with Nginx serving a static page, secured with a scoped-down Security Group and firewalld, plus an Elastic IP so the address stays fixed across restarts. Natural next steps would be adding HTTPS with a real domain and Let's Encrypt, moving the HTML into a small CI/CD pipeline (GitHub Actions → SCP or S3 deploy), and swapping the manual Nginx restart for a systemd unit or Docker container so the service recovers on its own after a reboot.
