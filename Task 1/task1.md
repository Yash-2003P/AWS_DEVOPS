# Task 1 — Create an AWS EC2 Instance (20 Marks)

## Objective

Launch an Ubuntu EC2 instance, create a Security Group, allow ports **22** and **80**, and connect using SSH.

## Deliverables

- EC2 Public IP
- EC2 Dashboard Screenshot
- SSH Login Screenshot

---

# Solution

## Step 1 — IAM Setup (Done Using Root Account)

AWS best practice recommends avoiding day-to-day work from the root account.

Actions performed:

- Logged into the AWS Root Account.
- Opened the IAM Console.
- Created an IAM Group named `aws_devops_group`.
- Created an IAM User named `aws_devops_intern`.
- Added the IAM user to the group.
- Attached the **AdministratorAccess** policy.

> **Note**
>
> `AdministratorAccess` was used only because this is a practice assignment.
>
> For production environments, the principle of least privilege should always be followed.
>
> For this assignment, `AmazonEC2FullAccess` would have been sufficient because it grants permissions required for launching and managing EC2 resources without providing unrestricted AWS access.

---

## Step 2 — Launch the EC2 Instance

Configuration used:

| Setting | Value |
|----------|-------|
| Name | `RHEL10` |
| AMI | Red Hat Enterprise Linux 10 |
| Instance Type | `t2.micro` |
| Key Pair | New `.pem` key |
| Network | Default VPC |
| Public IP | Enabled |
| Storage | Default EBS |

---

## Step 3 — Configure the Security Group

Inbound Rules:

| Port | Protocol | Source | Purpose |
|------|----------|---------|----------|
| 22 | SSH | My IP | Secure SSH Access |
| 80 | HTTP | Anywhere (`0.0.0.0/0`) | Public Website |

HTTPS (443) was intentionally not enabled because SSL was not required for this assignment.

---

## Step 4 — Launch the Instance

- Clicked **Launch Instance**.
- Waited until the instance status became **Running**.
- Waited for **2/2 Status Checks Passed**.
- Verified the attached Security Group.

---

## Step 5 — Connect Using SSH

Check the local operating system:

```bash
cat /etc/os-release
```

Move the downloaded PEM key into the WSL home directory:

```bash
sudo mv /mnt/c/Users/vish9/Downloads/rhel10_keys.pem /home/buntu1/
```

Verify the file exists:

```bash
ls -l rhel10_keys.pem
```

Set the required permissions:

```bash
chmod 400 rhel10_keys.pem
```

Verify the permissions:

```bash
ls -l rhel10_keys.pem
```

Connect to the EC2 instance:

```bash
ssh -i rhel10_keys.pem ec2-user@<ec_public_ip>
```

The username for Red Hat Enterprise Linux AMIs is:

```text
ec2-user
```

---

## Public IP Change During the Assignment

AWS assigns a dynamic public IP by default.

- First Public IP: `<ec_public_ip>`
- Second Public IP: `<ec_public_ip>`
- Final Elastic IP: `<ec2_elastic_public_ip>`

After stopping and starting the instance, AWS assigned a new public IP because an Elastic IP had not yet been attached.

---

## Time Taken

**1:38 PM – 1:52 PM (~14 minutes)**

Most of the time was spent waiting for the EC2 instance to complete initialization and pass its health checks.

---

## Problems Faced

### 1. SSH Refused the Private Key

Error:

```text
UNPROTECTED PRIVATE KEY FILE
```

Resolution:

```bash
chmod 400 rhel10_keys.pem
```

---

### 2. SSH Failed After Restarting the Instance

Cause:

AWS assigned a new dynamic public IP after the instance was stopped and started.

Resolution:

Updated the SSH command with the newly assigned public IP.

---

## Key Learnings

- Never perform routine AWS operations using the Root Account.
- Prefer least-privilege IAM policies whenever possible.
- `AmazonEC2FullAccess` is sufficient for EC2 management.
- RHEL EC2 instances use the `ec2-user` login account.
- Public IPs are dynamic unless an Elastic IP is attached.
- SSH requires the private key permissions to be set to `400`.
