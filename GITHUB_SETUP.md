# How to Get This on GitHub

Step by step from your computer right now.

---

## Step 1 — Create a GitHub account

Go to github.com and create an account if you don't have one.  
Username matters — use something professional. Your real name or a clean handle you'd put on a resume.

---

## Step 2 — Create a new repository

1. Click the **+** icon top right → **New repository**
2. Repository name: `offensive-security-journey` or `pentest-learning` or similar
3. Description: `Offensive security learning journey — penetration testing and AI red teaming documentation`
4. Set to **Public** — this is your portfolio, it needs to be visible
5. Do NOT initialize with README (you already have one)
6. Click **Create repository**

---

## Step 3 — Install Git on your computer

**Windows:**  
Download from git-scm.com and install with defaults

**Mac:**  
```bash
xcode-select --install
```

**Linux:**  
```bash
sudo apt install git
```

---

## Step 4 — Configure Git

Open terminal / command prompt:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

---

## Step 5 — Push this repository

Navigate to where you downloaded the pentest-journey folder, then:

```bash
cd pentest-journey

git init
git add .
git commit -m "Initial commit — learning journey structure"
git branch -M main
git remote add origin https://github.com/YOURUSERNAME/YOURREPONAME.git
git push -u origin main
```

GitHub will ask for your username and password.  
**Note:** GitHub no longer accepts passwords — use a Personal Access Token instead.  
Go to: github.com → Settings → Developer settings → Personal access tokens → Generate new token  
Use that token as your password when prompted.

---

## Step 6 — Update as you go

Every time you complete a room, machine, or AI finding:

```bash
git add .
git commit -m "Add writeup: [Room/Machine name]"
git push
```

That's it. Your portfolio updates publicly every time you push.

---

## Step 7 — Link your profile everywhere

Once the repo is live:
- Add the GitHub link to your LinkedIn profile under Featured or Projects
- Add it to your resume under Projects or Portfolio
- Link it in your TryHackMe profile bio
- Reference it in job applications

---

## Keeping it updated

The repo is only valuable if it reflects active, current work.  
A commit history showing consistent weekly activity is itself a signal to employers.  
Irregular bursts followed by long silences tell a different story than steady weekly progress.

You don't need to write a novel for each commit.  
A 10-line writeup pushed consistently beats a perfect writeup that never gets written.
