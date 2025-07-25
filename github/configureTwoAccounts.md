To set up **two GitHub accounts on Windows**, you need to manage two different SSH keys and configure Git to use the appropriate identity per repository. Here's a complete step-by-step guide:

---

## ✅ Step 1: Generate SSH Keys for Both Accounts

Open Git Bash or PowerShell:

### 🔹 Account 1 (Personal)

```bash
ssh-keygen -t rsa -b 4096 -C "personal@example.com"
# Save as: /c/Users/<your-username>/.ssh/id_rsa_personal
```

### 🔹 Account 2 (Work)

```bash
ssh-keygen -t rsa -b 4096 -C "work@example.com"
# Save as: /c/Users/<your-username>/.ssh/id_rsa_work
```

---

## ✅ Step 2: Add Keys to SSH Agent

Start SSH agent and add both keys:

```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa_personal
ssh-add ~/.ssh/id_rsa_work
```

---

## ✅ Step 3: Add Public Keys to GitHub

Copy the public key content and add it to each GitHub account:

```bash
cat ~/.ssh/id_rsa_personal.pub
cat ~/.ssh/id_rsa_work.pub
```

Add them via GitHub → **Settings → SSH and GPG keys → New SSH key**

---

## ✅ Step 4: Create SSH Config File

Create or edit your SSH config file:

```bash
nano ~/.ssh/config
```

Add this content:

```ssh
# Personal GitHub
Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_personal

# Work GitHub
Host github.com-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_work
```

---

## ✅ Step 5: Clone Repos Using Correct Host Alias

Use the correct alias in clone URL:

* **Personal**:

```bash
git clone git@github.com-personal:your-username/personal-repo.git
```

* **Work**:

```bash
git clone git@github.com-work:your-org/work-repo.git
```

---

## ✅ Step 6: Set Git Identity per Repository

Go into each repo and set local user config:

### 🔹 Personal Repo

```bash
git config user.name "Your Name"
git config user.email "personal@example.com"
```

### 🔹 Work Repo

```bash
git config user.name "Your Work Name"
git config user.email "work@example.com"
```

---

## 🧠 Optional: Use Credential Helper (HTTPS Only)

If you're using HTTPS (instead of SSH), you can set Git credentials per repository using:

```bash
git config credential.helper store
```

But SSH is strongly recommended for managing multiple accounts.

---

