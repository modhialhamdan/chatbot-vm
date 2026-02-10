# HOW TO CONNECT TO THE VM?
This guide explains everything end-to-end:
    1- Install required tools
    2- VPN access
    3- VM login via PuTTY
    4- SSH keys (VM access + Gitea access)
    5- Gitea repo clone
    6- VS Code Remote SSH for development

# STEP 1: Install Required Tools
## 1.1 Downloading VS 
Download it from the shared zipped file by Eng. Osamah.
Then install the VS Code extension:
Remote - SSH
## 1.2 Downloading PuTTY + FortiClient 
Open a ServiceNow ticket to request installation from the IT team:
    -PuTTY
    -FortiClient VPN
## 1.3 Downloading FortiToken on your mobile
Install FortiToken on your mobile phone.

# STEP 2: Get Access
You will meet Abdulmajeed.

You will have:

    - FortiClinet VPN Connection.
    - VM access details:
        -VM IP
        -Username
        -Initial password
    These will be provided from Abdulmajeed
    As: 
        Username: user.t
        VPN: aaaa93728shjakd28h
        IDM: kjWFHIJBDSJ43uhjdj

Important notes: 
    -VPN must be connected before you can reach the VM.
    -VPN Connection will disconnect each time you laptop turns off or sleep.


# STEP 3: First-Time VM Login (PuTTY)
## 3.1 Open PuTTY
Session : 

    Host Name: <10.106.120.4> (replace 4 with your number)
    Port: 22
    Connection type: SSH

## 3.2 First-time host key prompt

First time you connect, PuTTY may show a warning about the server key fingerprint.

Click Accept / Yes (if you are sure the IP is correct and given by the company).

## 3.3 Login using the initial password

PuTTY will ask for a password (first time only):

Password: <initial-password>

Now you are inside the Linux VM.

# STEP 4: Create SSH Keys (Inside the VM)
Best practice: Generate SSH keys inside the VM so secrets never leave the server environment.

## 4.1 Generate the SSH key
Run this inside the VM:

ssh-keygen -t ed25519 -C "vm-access"

Press Enter for default file location

Press Enter twice if you want no passphrase.

Expected key path:

~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub

## 4.2 Confirm keys exist

ls -la ~/.ssh


# STEP 5: Enable Passwordless Login to the VM
This step is only needed if your team requires passwordless SSH for repeated access.
Some companies keep password login enabled; others enforce keys.

## 5.1 Create authorized_keys

Inside the VM:

mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys

## 5.2 Add your public key

Copy the VM public key:

cat ~/.ssh/id_ed25519.pub

Paste it into:

~/.ssh/authorized_keys

Save:

Ctrl + O → Enter
Ctrl + X

## 5.3 Fix permissions

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

Common mistakes:

File name must be authorized_keys.

Permissions must not be too open.

# STEP 6: Create Your Gitea Account & Get Repo Access

## 6.1 Open the company Gitea website

## 6.2 Create your account

## 6.3 Ask the access manager Eng. Osamah to grant access to the org/repo

You must have:
Permission to read repo
Permission to push 

# STEP 7: Add the VM SSH Key to Gitea

## 7.1 Copy VM public key
Inside the VM:

cat ~/.ssh/id_ed25519.pub

## 7.2 Add it to Gitea
From inside the VM, copy the public key.
Then
In Gitea:

    User Settings → SSH Keys → Add Key
    Title: vm-<your-username>
    Paste the key
    Save

Now the VM can authenticate to Gitea via SSH.

# STEP 8: Configure SSH for Gitea (VM Side)

This makes sure the VM always uses the correct key when connecting to Gitea.

## 8.1 Create/Edit SSH config

Inside the VM:

nano ~/.ssh/config

Add:

Host gitea
  HostName <GITEA_IP_OR_DOMAIN>
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes

## 8.2 Fix permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config


Why this matters:

Without config, Git may ask for a password or use the wrong key.

# STEP 9: Test Gitea SSH Connection

Inside the VM:

ssh -T gitea


First time you may see:

Are you sure you want to continue connecting (yes/no)?


Type:

yes


Expected success output:

You've successfully authenticated, but Gitea does not provide shell access.


# STEP 10: Clone the Chatbot Repository
Inside the VM:

mkdir -p ~/dev
cd ~/dev
git clone git@gitea:<USER>@<key>/enterprise_chatbot.git

If it clones without password prompt → Everything is configured correctly.

# STEP 11: Configure Git Identity (Required for Commits)
Inside the VM:

git config --global user.name "<your-name>"
git config --global user.email "<your-email>"


# STEP 12: Open Project in VS Code (Remote SSH)
## 12.1 Connect VS Code to the VM

On Windows (local machine), open VS Code:

Press:

Ctrl + Shift + P


Run:

Remote-SSH: Connect to Host...


Enter:

ssh <your-username>@<VM_IP>


Select:

Linux

Accept / Trust host

VS Code will open a new window:

SSH: <VM_IP>  (or SSH: vm)

## 12.2 Open the project folder

In the VS Code remote window:

File → Open Folder
/home/<your-username>/dev/enterprise_chatbot

Now you are editing the project directly on the VM.

# Troubleshooting (Common Issues)
## 1) Git asks for password

If you see:

(git@<gitea>) Password:


Fix by ensuring:

SSH key is added in Gitea

VM has correct ~/.ssh/config for Gitea

Run:

ssh -T gitea

## 2) Bad owner or permissions on ~/.ssh/config

Fix:

chmod 700 ~/.ssh
chmod 600 ~/.ssh/config

## 3) Wrong authorized_keys name

Make sure it is:

authorized_keys


Not:

authorized_key

## 4) VS Code opens only a terminal (not remote dev)

You must connect using:

Remote-SSH: Connect to Host...


Not just “open terminal”.