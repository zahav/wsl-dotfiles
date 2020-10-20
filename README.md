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

# Install tools
sudo apt update && sudo apt upgrade
sudo apt install -y \
    containerd.io \
    docker-ce \
    docker-ce-cli \
    git \
    gnupg

# Add user to docker group
sudo usermod -aG docker $USER
```