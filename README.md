My Windows Subsystem for Linux Setup & Dotfiles
===============================================

Install
-------

### On WSL 2

#### Install dependencies

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
    git \
    gnupg \
    nodejs \
    yarn

# Add user to docker group
sudo usermod -aG docker $USER

# Test the installation
docker run --rm hello-world
```

#### Restore (or generate) the GPG key

- On old system, create a backup of a GPG key
  - `gpg --list-secret-keys`
  - `gpg --export-secret-keys {{KEY_ID}} > /tmp/private.key`
- On new system, import the key:
  - `gpg --import /tmp/private.key`
- Delete the `/tmp/private.key` on both sides

#### Setup Git

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
git config --global core.excludesfile ~/.gitignore_global

# Generate a new key
ssh-keygen -t rsa -b 4096 -C "${email}"

# Start ssh-agent and add the key to it
eval $(ssh-agent -s)
ssh-add ~/.ssh/github_rsa

# Display the public key ready to be copy pasted to GitHub
cat ~/.ssh/github_rsa.pub
```

- [Add the generated key to GitHub](https://github.com/settings/ssh/new)