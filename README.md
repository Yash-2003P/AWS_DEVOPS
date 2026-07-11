# AWS DevOps Engineer Intern Assignment (1 Day)

This repo has the full work for deploying a simple website on an AWS EC2 instance (RHEL 10) using Nginx.

## Files In This Repo

| File | What's inside |
|---|---|
| `task1.md` | How I created the EC2 instance, IAM setup, Security Group, SSH connection |
| `task2.md` | How I installed and managed Nginx, monitoring commands |
| `task3.md` | How I replaced the default Nginx page with my own website |
| `task4.md` | How I set up this GitHub repo |
| `task5.md` | Points to the full report (`AWS-DevOps-Assignment-Report.docx`) |
| `index.html` | The actual website file running on the EC2 instance |
| `AWS-DevOps-Assignment-Report.docx` | Full report — AWS services, all commands, problems faced, learnings, time taken |

## Short Summary Of What I Did

1. **IAM first, not root.** Created IAM group `aws_devops_group` and IAM user `aws_devops_intern` from the root account, gave the group `AdministratorAccess` (only for this demo — `AmazonEC2FullAccess` alone is enough for this exact task). Did all further work logged in as this IAM user.
2. **Launched EC2 instance** — AMI: Red Hat Enterprise Linux 10, type: `t2.micro`, default VPC, default EBS volume.
3. **Security Group** — opened SSH (22) to My IP only, HTTP (80) to everyone (0.0.0.0/0).
4. **Connected with SSH** using the `.pem` key from WSL2 (`ssh -i rhel10_keys.pem ec2-user@<EC2_PUBLIC_IP>`).
5. **Installed Nginx** with `dnf`, opened port 80 on `firewalld` (OS-level firewall, separate from AWS Security Group), checked SELinux (`getenforce` → `Enforcing`, left as default).
6. **Enabled and started Nginx**, checked status, restarted it, checked disk (`df -h`), memory (`free -h`), and running processes (`ps aux`, `top`).
7. **Replaced the default Nginx page** with my own `index.html` (Name, College, Branch, Email, Date) using `cp` (not `mv`) plus `restorecon` to keep the correct SELinux file label.
8. **Uploaded everything to GitHub** through the website UI (no local `git` CLI used).
9. **Bonus — Elastic IP.** Allocated an Elastic IP and attached it to the instance, so the public IP stays fixed from now on.

## Public IP Changed During This Assignment — Important Note

I did not use Elastic IP from the start, so my EC2 public IP was dynamic (changes on every stop/start). Over this assignment I actually had **3 different public IPs**, in this order:
1. First IP — assigned when instance first launched.
2. Second IP — a new one got assigned automatically after I stopped the instance (ahead of a power cut at my place) and started it again later. The old SSH command just stopped connecting at this point — this is normal AWS behaviour, not an error.
3. Final IP — the Elastic IP, allocated in the bonus task. This one does not change anymore.

In the markdown files here, the first two are written as `<EC2_PUBLIC_IP>` and the final fixed one as `<EC2_ELASTIC_PUBLIC_IP>`. Actual IP numbers are written in the DOCX report.

## Live Site
`http://<EC2_ELASTIC_PUBLIC_IP>` — see the DOCX report for the actual current IP.

## About Screenshots
Screenshots (EC2 dashboard, SSH login, `systemctl status`, website in browser, Elastic IP) are **not** in this GitHub repo and **not** inside the DOCX file. They go only inside the final submission ZIP folder, as separate image files, since that is what the assignment submission checklist asks for.
