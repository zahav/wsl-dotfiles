My Windows Subsystem for Linux Setup & Dotfiles
===============================================

Install
-------

### On Windows

#### 1. Enable WSL2

Run in PowerShell, as admin (elevated):

```ps1
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
wsl --set-default-version 2
```

- [Install Ubuntu from Microsoft Store](https://www.microsoft.com/en-au/p/ubuntu/9nblggh4msv6)

### On WSL 2

#### 2. Install dependencies

```bash
# Use apt over HTTPS
sudo apt update && sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# Add Docker to sources.list
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Add Node.js to sources.list
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

# Add Yarn to sources.list
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

# Install development tools
sudo apt update && sudo apt upgrade
sudo apt install -y \
    containerd.io \
    docker-ce \
    docker-ce-cli \
    fontconfig \
    git \
    gnupg \
    nodejs \
    unzip \
    yarn \
    zsh

# Add user to docker group
sudo usermod -aG docker $USER

# Test the installation
docker run --rm hello-world
```

#### 3. Restore (or generate) the GPG key

- On old system, create a backup of a GPG key
  - `gpg --list-secret-keys`
  - `gpg --export-secret-keys {{KEY_ID}} > /tmp/private.key`
- On new system, import the key:
  - `gpg --import /tmp/private.key`
- Delete the `/tmp/private.key` on both sides

#### 4. Setup Git

```bash
# Set the git author email and username
email="EMAIL"
username="USERNAME"
gpgkeyid="KEY_ID"

# Configure Git
git config --global user.email "${email}"
git config --global user.name "${username}"
git config --global user.signingkey "${gpgkeyid}"
git config --global commit.gpgsign true
git config --global core.excludesfile ~/.gitignore

# Generate a new key
ssh-keygen -t rsa -b 4096 -C "${email}"

# Start ssh-agent and add the key to it
eval $(ssh-agent -s)
ssh-add ~/.ssh/github_rsa

# Display the public key ready to be copy pasted to GitHub
cat ~/.ssh/github_rsa.pub
```

- [Add the generated key to GitHub](https://github.com/settings/ssh/new)

#### 5. Setup zsh

```bash
# Change the default shell to ZSH
chsh -s /usr/bin/zsh

# Create a Code directory for all our work
mkdir ~/Code

# Finally clone the repository
mkdir -p ~/.dotfiles
git clone git@github.com:zahav/wsl-dotfiles.git ~/.dotfiles

# Link custom dotfiles
ln -sf ~/.dotfiles/.gitignore_global ~/.gitignore
```

#### 6. Customise our fonts

```bash
# Install JetBrains Mono typeface
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/JetBrains/JetBrainsMono/master/install_manual.sh)"
```

##### Alternate fonts to try

- Cascadia Code
- DejaVu Sans
- FuraMono
- FiraCode
- Hack
- Menlo
