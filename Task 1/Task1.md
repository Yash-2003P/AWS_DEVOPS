# Task 1 — Create an AWS EC2 Instance (20 Marks)
# Launch an Ubuntu EC2 instance, create a Security Group, allow ports 22 and 80, and connect using SSH.
# Deliverables: EC2 Public IP, EC2 Dashboard screenshot, SSH login screenshot.

## Solution — What I Did Step by Step

### Step 1 — IAM Setup (Done from Root Account)

I did not do any EC2 work as root. Root was used only one time to set up IAM.

- Logged into AWS root account.
- Went to **IAM** service in AWS console.
- Created an IAM group named `aws_devops_group`.
- Created an IAM user named `aws_devops_intern`.
- Added user `aws_devops_intern` to group `aws_devops_group`.
- Attached policy **`AdministratorAccess`** to the group.
  - **Note:** I used `AdministratorAccess` here only because this is a demo/practice assignment. In real work you should not give full admin rights just to launch one EC2 instance. The correct and safer policy for this exact task is **`AmazonEC2FullAccess`** — it gives full access to only EC2 (launch, stop, terminate, security groups, key pairs, EBS volumes) and nothing else. So this entire task can be done using only `AmazonEC2FullAccess` instead of `AdministratorAccess`.
- Logged out of root account.
- Logged back in using the IAM user `aws_devops_intern` credentials.
- All EC2 work below was done using this IAM user, not root.

### Step 2 — Launch EC2 Instance

- EC2 Console → Instances → **Launch Instance**.
- Name: `RHEL10`.
- AMI: Searched "Red Hat" in AMI search box, selected **Red Hat Enterprise Linux 10** (official Red Hat AMI).
- Instance type: `t2.micro` (free tier eligible).
- Key pair: Created new key pair, type `.pem`, downloaded it.
- Network settings: Left as default — default VPC, default subnet, auto-assign public IP = enabled.
- Storage: Default EBS volume, no changes made.

### Step 3 — Security Group Inbound Rules

Created a new Security Group while launching. Rules I added:

- **SSH (Port 22)** — Source: My IP (only I can SSH in, safer than opening to everyone).
- **HTTP (Port 80)** — Source: Anywhere-IPv4 (`0.0.0.0/0`) because the website must be reachable by anyone on the internet.
- HTTPS (443) was **not** opened — not needed since there is no SSL certificate for this task.

### Step 4 — Launch and Verify

- Clicked **Launch Instance**.
- Waited for status checks to show "2/2 checks passed". This took a few minutes because the instance needs time to boot up and pass health checks — it is not instant.
- Confirmed instance state = `running`.
- Went to instance → **Security** tab → double-checked Security Group rules are correct.

### Step 5 — Connect via SSH (from WSL2 Ubuntu)

The `.pem` key file downloads to Windows Downloads folder. Since I use WSL2, I first moved it into the WSL2 home directory:

```bash
cat /etc/os-release
```
- Confirms the OS info on my local WSL2 machine.

```bash
sudo mv /mnt/c/Users/vish9/Downloads/rhel10_keys.pem /home/buntu1/
```
- Moves the `.pem` file from Windows Downloads to WSL2 home folder.

```bash
ls -l rhel10_keys.pem
```
- Checks the file exists and shows its current permissions.

```bash
chmod 400 rhel10_keys.pem
```
- Changes permission to read-only for owner only. SSH will refuse to use a `.pem` file if other users can read it.

```bash
ls -l rhel10_keys.pem
```
- Verifies the permission is now `-r--------` (400).

```bash
ssh -i rhel10_keys.pem ec2-user@<ec_public_ip>
```
- Connects to the EC2 instance using the key file. Username for RHEL AMIs is `ec2-user` (not `ubuntu`).
- Typed `yes` when asked about host fingerprint (first time only).

### Public IP Changed During This Assignment

I did not use Elastic IP at the start. AWS gives a dynamic public IP by default. This IP changes every time you stop and start the instance.

- **First IP** (`<ec_public_ip>`) — assigned when instance first launched.
- **Second IP** (`<ec_public_ip>`) — new IP assigned automatically after I stopped the instance (before a power cut at my place) and started it again later. Old SSH command stopped connecting.
- **Final IP** (`<ec2_elastic_public_ip>`) — the Elastic IP, allocated in the bonus task. This one is fixed and does not change.

**Screenshot taken:** EC2 dashboard showing the running instance.

## Time Taken

**1:38 PM – 1:52 PM (~14 minutes)**

Most of this was waiting for the EC2 instance to initialize and pass its status checks before SSH would connect. Not extra work, just boot wait time.

## Problems I Faced

1. **Key file permission too open.** The `.pem` file downloads with `644` permission. SSH refused to connect with error `UNPROTECTED PRIVATE KEY FILE`. Fixed by running `chmod 400 rhel10_keys.pem` before the first SSH attempt.

2. **Old SSH command stopped working after power cut.** I had stopped the instance before power went off. When I restarted after power came back, the old IP did not connect. I first thought the instance broke. Then I checked the EC2 console and saw the public IP was different — this is normal AWS behaviour for non-Elastic IPs, not an error. Updated my SSH command with the new IP and it worked.

## What I Learned

- Never do daily work as root — always create an IAM user with least-privilege permissions.
- `AdministratorAccess` vs `AmazonEC2FullAccess` — for a single EC2 task, `AmazonEC2FullAccess` is enough and safer.
- RHEL AMI login user is `ec2-user`, not `ubuntu`.
- EC2 public IP is dynamic by default — it changes on every stop/start unless you attach an Elastic IP.
- `.pem` key must be `chmod 400` before first SSH connection, or SSH will refuse.
