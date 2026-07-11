# Bonus: Attach an Elastic IP (+10 Marks)

Complete any ONE: Attach an Elastic IP, create another Security Group, install Docker and run hello-world, or create a shell script to restart Nginx.

---

## What I Did — Attached an Elastic IP

I chose to attach an Elastic IP to my EC2 instance. By default, EC2 gives a dynamic public IP that changes every time you stop and start the instance. An Elastic IP is a fixed public IP that stays the same even after stop/start.

### Step 1 — Allocated the Elastic IP

- Went to EC2 Console → **Elastic IPs** (under Network & Security section in the left sidebar).
- Clicked **"Allocate Elastic IP address"**.
- Left the network scope as default, clicked **Allocate**.
- A new Elastic IP was allocated and showed up in the list.

### Step 2 — Associated the Elastic IP With My Instance

- Selected the newly allocated Elastic IP from the list.
- Clicked **Actions** → **"Associate Elastic IP address"**.
- In the dropdown, chose my running EC2 instance (`RHEL10`).
- Clicked **Associate**.
- The Elastic IP was now linked to my instance. The old dynamic public IP was released by AWS automatically.

### Step 3 — Connected via SSH Using the New Elastic IP

```bash
ssh -i rhel10_keys.pem ec2-user@<ec2_elastic_public_ip>
```
- Connected to the instance using the Elastic IP. The key file and username are the same, only the IP changed.

### Step 4 — Tested the Website on the New IP

```bash
curl http://<ec2_elastic_public_ip>
```
- This did **not** return my website page. The page was still being served on the old dynamic IP internally, so the website was not accessible through the Elastic IP yet.

### Step 5 — Restarted Nginx

```bash
sudo systemctl restart nginx
```
- Restarted the nginx service so it picks up the new network interface (the Elastic IP). After the restart, nginx started listening on the new IP as well.

### Step 6 — Verified the Website on the Elastic IP

```bash
curl http://<ec2_elastic_public_ip>
```
- Now the website loaded correctly through the Elastic IP. The response showed my HTML page with name, college, branch, email, and date.

### Step 7 — Also Tested File Transfer With the New IP

```bash
scp -i rhel10_keys.pem ec2-user@<ec2_elastic_public_ip>:/home/ec2-user/abc.txt /mnt/c
```
- Copied a test file from the EC2 instance to my local machine using the Elastic IP. Worked correctly.

## Time Taken

**4:57 PM – 5:11 PM (~14 minutes)**

This was done after a power cut at my place (~4:40 PM). When the power came back, I restarted the instance, saw the public IP had changed (again), and decided to fix this permanently with an Elastic IP.

## Problems I Faced

**Website not loading on the Elastic IP after association.**

After associating the Elastic IP, I curled the new IP and did not get my page. The nginx web server was still bound to the old dynamic IP internally. Restarting nginx fixed this — `sudo systemctl restart nginx` made nginx rebind to all available network interfaces including the new Elastic IP.

## What I Learned

- Elastic IP allocation and association is done entirely from the AWS Console, no CLI or server-side commands needed.
- After associating a new Elastic IP, the web server (nginx) may need a restart to start serving on the new IP.
- Elastic IP is free while attached to a running instance. AWS charges a small hourly fee if it is allocated but not attached to a running instance. Release it after submitting the assignment to avoid charges.