# Task 4: Git & GitHub (20 Marks)

Create a GitHub repository and upload the HTML file and README.md. Include EC2 setup steps, Nginx installation, and commands used.
Deliverables: GitHub repository link.

---

## Solution — What I Did Step by Step

### Step 1 — Created the GitHub Repository

- Went to **github.com** in my browser.
- Clicked **"New repository"**.
- Set the repository to **public**.
- Gave it a name and description related to the assignment.
- Clicked **"Create repository"**.

### Step 2 — Created Folder Structure Inside the Repository

Instead of putting all files flat in the root, I organised them into separate folders by task. On GitHub I clicked **"Create new file"** and for each folder I typed the folder name followed by `/` (e.g. `Task 1/`) which GitHub treats as creating a folder. Then inside each folder I created the corresponding file.

### Step 3 — Uploaded Files Using GitHub Website UI

I did the Git/GitHub work using the GitHub website UI directly, not `git` CLI on my local machine. For each folder and file I used **"Add file → Create new file"** (for new files) and **"Add file → Upload files"** (for existing files like `index.html`).

**Files uploaded:**

- `Task 1/Task1.md` — EC2 instance creation, IAM setup, Security Group, SSH connection.
- `Task 2/Task2.md` — Nginx installation, firewalld configuration, monitoring commands.
- `Task 3/Task3.md` — Website hosting, HTML creation, SELinux 403 fix.
- `Task 3/index.html` — the HTML file that is running on the EC2 instance.
- `Task 4/Task4.md` — this file (Git & GitHub steps).
- `Task 5/Task5.md` — documentation task (PDF report details).
- `Bonus/Bonus.md` — Elastic IP bonus task.
- `README.md` — main README with EC2 setup steps, Nginx installation, and all commands used.

### Step 4 — Git Commands Used (Reference — What Pushing/Committing Looks Like)

Although I used the GitHub UI for this assignment, these are the equivalent `git` commands for creating a repo, adding files, committing, and pushing:

```bash
cd ~/my-project
git init
```
- Initialises a new git repository in the current directory.

```bash
git branch -M main
```
- Renames the default branch to `main`.

```bash
git add .
```
- Stages all files and folders for commit. The `.` means everything in the current directory (including subdirectories like `Task 1/`, `Task 2/`, etc.).

```bash
git status
```
- Shows which files are staged (green), unstaged (red), and untracked. Good to run before committing so you know exactly what is going in.

```bash
git commit -m "Initial commit: AWS DevOps assignment — EC2 setup, Nginx, website hosting"
```
- Commits the staged files with a descriptive message explaining what is in this commit.

```bash
git remote add origin https://github.com/<your-username>/<repo-name>.git
```
- Links the local repository to the remote GitHub repository. `origin` is the default remote name.

```bash
git push -u origin main
```
- Pushes the commit to GitHub. `-u` sets the upstream branch so future pushes only need `git push`.

### Important Note About GitHub Links

- `github.com/<user>/<repo>/blob/main/file.html` — This is the **viewer page**. It shows GitHub's own HTML around your file. If you `curl` or `wget` this link, you get GitHub's page, not your file.
- `raw.githubusercontent.com/<user>/<repo>/main/file.html` — This gives the **actual raw file**. Use this link when you want to download the file content.

## Repository Link

`https://github.com/<your-username>/<repo-name>`

## Time Taken

**~3:00 PM** — Repository was created, folders were made, and all files were uploaded through the browser UI. This was quick since I used the "Add file" button on GitHub for everything.

## What I Learned

- A GitHub repository can be fully set up using just the website UI, no local `git` CLI needed.
- GitHub lets you create folders by typing `folder-name/` when creating a new file — the `/` at the end tells GitHub to make a folder.
- Difference between a `github.com/.../blob/...` link (viewer page) and a `raw.githubusercontent.com` link (actual file content).
- `git add .` stages everything including files inside subdirectories, which is useful when you have a folder-based structure.