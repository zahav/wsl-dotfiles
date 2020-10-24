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

# Add AzureCLI to sources.list
curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" |
    sudo tee /etc/apt/sources.list.d/azure-cli.list

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
    azure-cli \
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
git config --global core.excludesfile $HOME/.gitignore

# Create a Code directory for all our work
mkdir $HOME/Code

# Generate a new key
ssh-keygen -t rsa -b 4096 -C "${email}"

# Start ssh-agent and add the key to it
eval $(ssh-agent -s)
ssh-add $HOME/.ssh/github_rsa

# Display the public key ready to be copy pasted to GitHub
cat $HOME/.ssh/github_rsa.pub
```

- [Add the generated key to GitHub](https://github.com/settings/ssh/new)

#### 5. Setup ZSH and Oh My Zsh

```bash
# Change the default shell to ZSH
chsh -s $(which zsh)

# Install Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Finally clone the repository
mkdir -p $HOME/.dotfiles
git clone git@github.com:zahav/wsl-dotfiles.git $HOME/.dotfiles

# Removes .zshrc from $HOME (if it exists) and symlinks the .zshrc file from the .dotfiles
rm $HOME/.zshrc
ln -s $HOME/.dotfiles/.zshrc $HOME/.zshrc

# Link custom dotfiles
ln -s $HOME/.dotfiles/.aliases.zsh $HOME/.aliases.zsh
ln -s $HOME/.dotfiles/.gitignore_global $HOME/.gitignore
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

#### 7. Copy useful files to Windows

```bash
windowsUserProfile=/mnt/c/Users/$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')

# Windows Terminal settings
cp $HOME/.dotfiles/terminal-settings.json ${windowsUserProfile}/AppData/Local/Packages/Microsoft.WindowsTerminal_8wekyb3d8bbwe/LocalState/settings.json

# Avoid too much RAM consumption
cp $HOME/.dotfiles/.wslconfig ${windowsUserProfile}/.wslconfig
```