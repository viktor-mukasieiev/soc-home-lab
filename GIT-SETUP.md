# GitHub Setup Guide — macOS (Starting from Zero)

## Step 1: Create GitHub Account

1. Go to https://github.com
2. Click **Sign up**
3. Use a professional username — this will be visible to employers
   - Good: `firstname-lastname`, `firstnamelastname`, `fl-security`
   - Avoid: numbers, random strings, nicknames
4. Verify email

---

## Step 2: Install Homebrew (macOS package manager)

Open **Terminal** on macOS and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify:
```bash
brew --version
```

---

## Step 3: Install Git

```bash
brew install git
```

Verify:
```bash
git --version
```

---

## Step 4: Configure Git Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

> Use the same email as your GitHub account.

---

## Step 5: Authenticate GitHub from Terminal

```bash
brew install gh
gh auth login
```

Follow the prompts:
- Select: **GitHub.com**
- Select: **HTTPS**
- Select: **Login with a web browser**
- Copy the code shown, press Enter — browser opens, paste the code

Verify:
```bash
gh auth status
```

---

## Step 6: Create Repository on GitHub

```bash
gh repo create soc-home-lab --public --description "Home SOC lab — detection engineering, Splunk, Sysmon, Kali attack simulation"
```

---

## Step 7: Clone to macOS and Add Files

```bash
# Clone the empty repo
cd ~
git clone https://github.com/YOUR_USERNAME/soc-home-lab.git
cd soc-home-lab
```

Now copy all the downloaded files from this chat into the `soc-home-lab` folder, keeping the folder structure:

```
soc-home-lab/
├── README.md
├── GIT-SETUP.md
└── infrastructure/
    ├── mikrotik/
    │   ├── wireguard-vps-setup.md
    │   └── mullvad-vlan-setup.md
    └── network-diagram/
        └── architecture.md
```

---

## Step 8: First Commit and Push

```bash
git add .
git commit -m "Initial commit: network infrastructure documentation"
git push
```

Open `https://github.com/YOUR_USERNAME/soc-home-lab` — README should render immediately.

---

## Step 9: Add Screenshots (Optional — do after first push)

For each screenshot you want to add:

1. Create folder: `assets/screenshots/`
2. Name files descriptively: `mikrotik-wireguard-handshake.png`, `vlan-routing-table.png`
3. Reference in markdown: `![WireGuard Handshake](../../assets/screenshots/mikrotik-wireguard-handshake.png)`

---

## Future Workflow — Every Lab Session

```bash
cd ~/soc-home-lab
git add .
git commit -m "lab: brief description of what was done"
git push
```

> Rule: every completed lab task = one commit. Treat it like a professional work log.

---

## Commit Message Convention

| Prefix | Use for |
|--------|---------|
| `init:` | First setup of something |
| `infra:` | Network, VMs, infrastructure changes |
| `lab:` | Detection scenarios, attack simulations |
| `docs:` | Documentation updates |
| `fix:` | Corrections to existing configs |

Examples:
```
infra: add Sysmon config to Windows VM
lab: detect nmap scan with Splunk alert
docs: add screenshots to WireGuard setup
fix: correct mangle rule exclusion logic
```
