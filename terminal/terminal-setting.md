# Zsh Configuration for Cloud K8s Docker Development

This notebook will set up a powerful zsh environment optimized for Kubernetes, Docker, and cloud development.

## Prerequisites Check

First, let's check what's already installed on your system:

```bash { name=check-prerequisites }
echo "=== System Information ==="
uname -a
echo ""

echo "=== Checking existing tools ==="
echo "Zsh: $(which zsh 2>/dev/null || echo 'Not installed')"
echo "Git: $(which git 2>/dev/null || echo 'Not installed')"
echo "Curl: $(which curl 2>/dev/null || echo 'Not installed')"
echo "Docker: $(which docker 2>/dev/null || echo 'Not installed')"
echo "Kubectl: $(which kubectl 2>/dev/null || echo 'Not installed')"
echo "Helm: $(which helm 2>/dev/null || echo 'Not installed')"
```

## Fonts

```bash

# Install several popular Nerd Fonts
brew install --cask font-meslo-lg-nerd-font
brew install --cask font-fira-code-nerd-font
brew install --cask font-hack-nerd-font
brew install --cask font-source-code-pro-nerd-font

echo "Multiple Nerd Fonts installed! Choose 'MesloLGS Nerd Font' in your terminal."
```

## Network Troubleshooting (If Needed)

If you encounter network connectivity issues during installation, run this diagnostic:

```bash { name=network-diagnostics }
echo "=== Network Connectivity Diagnostics ==="
echo ""

echo "1. Testing basic internet connectivity..."
if ping -c 2 8.8.8.8 > /dev/null 2>&1; then
    echo "✓ Internet connection OK"
else
    echo "✗ No internet connection - check your network"
    echo "  Try: Reconnect WiFi, check VPN, restart network interface"
fi

echo ""
echo "2. Testing DNS resolution..."
if nslookup github.com > /dev/null 2>&1; then
    echo "✓ DNS resolution OK"
else
    echo "✗ DNS resolution failed - trying to fix..."
    echo "  Switching to Google DNS..."
    
    # Try to fix DNS (macOS)
    if [[ "$OSTYPE" == "darwin"* ]]; then
        sudo networksetup -setdnsservers "Wi-Fi" 8.8.8.8 8.8.4.4
        echo "  DNS switched to Google DNS (8.8.8.8, 8.8.4.4)"
    else
        echo "  Manually set DNS to 8.8.8.8 and 8.8.4.4 in your network settings"
    fi
fi

echo ""
echo "3. Testing GitHub accessibility..."
if curl -Is --connect-timeout 10 https://github.com > /dev/null 2>&1; then
    echo "✓ GitHub is accessible"
else
    echo "✗ GitHub not accessible"
    echo "  Possible solutions:"
    echo "  - Check if you're behind a corporate firewall"
    echo "  - Try disconnecting/reconnecting VPN"
    echo "  - Check GitHub status at: https://www.githubstatus.com/"
fi

echo ""
echo "4. Current DNS servers:"
if [[ "$OSTYPE" == "darwin"* ]]; then
    scutil --dns | grep 'nameserver\[0\]' | head -3
else
    cat /etc/resolv.conf | grep nameserver
fi
```

## Step 1: Install Zsh (if needed)

```bash { name=install-zsh }
# Detect OS and install zsh
if [[ "$OSTYPE" == "linux-gnu"* ]]; then
    if command -v apt-get >/dev/null; then
        sudo apt-get update
        sudo apt-get install -y zsh curl git
    elif command -v yum >/dev/null; then
        sudo yum install -y zsh curl git
    elif command -v pacman >/dev/null; then
        sudo pacman -S zsh curl git
    fi
elif [[ "$OSTYPE" == "darwin"* ]]; then
    # macOS - zsh is usually pre-installed
    if ! command -v brew >/dev/null; then
        echo "Installing Homebrew..."
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    fi
    brew install git curl
fi

echo "Zsh installation completed!"
```

## Step 2: Install Oh My Zsh Framework

```bash { name=install-oh-my-zsh }
# Install Oh My Zsh
if [ ! -d "$HOME/.oh-my-zsh" ]; then
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
    echo "Oh My Zsh installed successfully!"
else
    echo "Oh My Zsh is already installed"
fi
```

## Step 3: Install Powerlevel10k Theme

```bash { name=install-powerlevel10k }
# Install Powerlevel10k theme
P10K_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"

if [ ! -d "$P10K_DIR" ]; then
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "$P10K_DIR"
    echo "Powerlevel10k installed successfully!"
else
    echo "Powerlevel10k is already installed"
fi
```

## Step 4: Install Essential Zsh Plugins

```bash { name=install-zsh-plugins }
# Function to test GitHub connectivity
test_github() {
    curl -Is --connect-timeout 10 https://github.com > /dev/null 2>&1
}

echo "Testing GitHub connectivity..."
if test_github; then
    echo "✓ GitHub is accessible, proceeding with git clone..."
    
    # Install zsh-autosuggestions
    AUTOSUGGESTIONS_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
    if [ ! -d "$AUTOSUGGESTIONS_DIR" ]; then
        git clone https://github.com/zsh-users/zsh-autosuggestions "$AUTOSUGGESTIONS_DIR"
        echo "zsh-autosuggestions installed!"
    else
        echo "zsh-autosuggestions already installed"
    fi

    # Install zsh-syntax-highlighting
    HIGHLIGHTING_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
    if [ ! -d "$HIGHLIGHTING_DIR" ]; then
        git clone https://github.com/zsh-users/zsh-syntax-highlighting.git "$HIGHLIGHTING_DIR"
        echo "zsh-syntax-highlighting installed!"
    else
        echo "zsh-syntax-highlighting already installed"
    fi

else
    echo "⚠️  GitHub not accessible, using manual download method..."
    
    # Create plugin directories
    mkdir -p "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
    mkdir -p "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"

    # Download using curl instead of git clone
    echo "Downloading zsh-autosuggestions..."
    if curl -L https://github.com/zsh-users/zsh-autosuggestions/archive/master.tar.gz | tar -xz --strip-components=1 -C "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"; then
        echo "✓ zsh-autosuggestions downloaded successfully!"
    else
        echo "✗ Failed to download zsh-autosuggestions"
    fi

    echo "Downloading zsh-syntax-highlighting..."
    if curl -L https://github.com/zsh-users/zsh-syntax-highlighting/archive/master.tar.gz | tar -xz --strip-components=1 -C "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"; then
        echo "✓ zsh-syntax-highlighting downloaded successfully!"
    else
        echo "✗ Failed to download zsh-syntax-highlighting"
    fi
fi

# Install fzf (fuzzy finder)
if [ ! -d "$HOME/.fzf" ]; then
    echo "Installing fzf..."
    if test_github; then
        git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
        ~/.fzf/install --all
        echo "fzf installed successfully!"
    else
        echo "⚠️  Skipping fzf installation due to connectivity issues"
        echo "You can install it later with: git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install --all"
    fi
else
    echo "fzf is already installed"
fi


# Install zsh-syntax-highlighting
HIGHLIGHTING_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
if [ ! -d "$HIGHLIGHTING_DIR" ]; then
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git "$HIGHLIGHTING_DIR"
    echo "zsh-syntax-highlighting installed!"
else
    echo "zsh-syntax-highlighting already installed"
fi

# Install fzf (fuzzy finder)
if [ ! -d "$HOME/.fzf" ]; then
    git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
    ~/.fzf/install --all
    echo "fzf installed successfully!"
else
    echo "fzf is already installed"
fi
```

## Step 5: Create Advanced Zsh Configuration

```bash { name=create-zshrc, cwd=$HOME }
# Backup existing .zshrc if it exists
if [ -f "$HOME/.zshrc" ]; then
    cp "$HOME/.zshrc" "$HOME/.zshrc.backup.$(date +%Y%m%d_%H%M%S)"
    echo "Existing .zshrc backed up"
fi

# Create new .zshrc configuration
cat > "$HOME/.zshrc" << 'EOF'
# Path to oh-my-zsh installation
export ZSH="$HOME/.oh-my-zsh"

# Theme
ZSH_THEME="powerlevel10k/powerlevel10k"

plugins=(
  git docker docker-compose kubectl helm terraform aws azure gcloud dotnet
  zsh-autosuggestions zsh-syntax-highlighting fzf z colored-man-pages command-not-found
)

source $ZSH/oh-my-zsh.sh

# ============================================
# KUBERNETES ALIASES & FUNCTIONS
# ============================================

# Core kubectl aliases
alias k="kubectl"
alias kgp="kubectl get pods"
alias kgs="kubectl get svc"
alias kgd="kubectl get deployment"
alias kgn="kubectl get nodes"
alias kga="kubectl get all"
alias kd="kubectl describe"
alias kdel="kubectl delete"
alias kl="kubectl logs"
alias kex="kubectl exec -it"
alias kpf="kubectl port-forward"

# Context and namespace management
alias kctx="kubectl config current-context"
alias kns="kubectl config view --minify --output 'jsonpath={..namespace}'"
alias kcc="kubectl config use-context"
alias kcn="kubectl config set-context --current --namespace"

# Advanced aliases
alias kgpa="kubectl get pods --all-namespaces"
alias kgsca="kubectl get secret --all-namespaces"
alias kgcma="kubectl get configmap --all-namespaces"

# Watch commands
alias kwp="watch kubectl get pods"
alias kws="watch kubectl get svc"
alias kwd="watch kubectl get deployment"

# ============================================
# DOCKER ALIASES
# ============================================

alias d="docker"
alias dc="docker-compose"
alias dps="docker ps"
alias dpa="docker ps -a"
alias di="docker images"
alias dex="docker exec -it"
alias dl="docker logs"
alias dsp="docker system prune"
alias dspa="docker system prune -a"
alias dvp="docker volume prune"
alias dnp="docker network prune"

# Docker compose shortcuts
alias dcu="docker-compose up"
alias dcd="docker-compose down"
alias dcb="docker-compose build"
alias dcl="docker-compose logs"
alias dcps="docker-compose ps"

# ============================================
# HELM ALIASES
# ============================================

alias h="helm"
alias hls="helm list"
alias hla="helm list --all-namespaces"
alias hi="helm install"
alias hu="helm upgrade"
alias hd="helm delete"
alias hs="helm status"
alias hr="helm repo"

# ============================================
# CLOUD PROVIDER ALIASES
# ============================================

# AWS
alias awsp="aws sts get-caller-identity"
alias awsr="aws configure list"

# Azure
alias azl="az login"
alias azs="az account show"
alias azg="az group list"

# GCP
alias gcpl="gcloud auth list"
alias gcpp="gcloud config list project"
alias gcps="gcloud config set project"

# ============================================
# DEVELOPMENT ALIASES
# ============================================

# .NET
alias dn="dotnet"
alias dnb="dotnet build"
alias dnr="dotnet run"
alias dnt="dotnet test"
alias dnp="dotnet publish"
alias dnw="dotnet watch"

# Common development
alias ll="ls -la"
alias la="ls -la"
alias ..="cd .."
alias ...="cd ../.."
alias ....="cd ../../.."
alias grep="grep --color=auto"

# ============================================
# USEFUL FUNCTIONS
# ============================================

# Quick kubectl context switcher
kswitch() {
  if [ $# -eq 0 ]; then
    kubectl config get-contexts
  else
    kubectl config use-context $1
  fi
}

# Get pod logs with fuzzy search
klogs() {
  local pod=$(kubectl get pods --no-headers -o custom-columns=":metadata.name" | fzf)
  if [ -n "$pod" ]; then
    kubectl logs -f $pod
  fi
}

# Execute into pod with fuzzy search
kexec() {
  local pod=$(kubectl get pods --no-headers -o custom-columns=":metadata.name" | fzf)
  if [ -n "$pod" ]; then
    kubectl exec -it $pod -- /bin/bash
  fi
}

# Port forward with fuzzy search
kport() {
  local pod=$(kubectl get pods --no-headers -o custom-columns=":metadata.name" | fzf)
  if [ -n "$pod" ]; then
    echo "Enter local port:"
    read local_port
    echo "Enter remote port:"
    read remote_port
    kubectl port-forward $pod $local_port:$remote_port
  fi
}

# Docker container shell access
dshell() {
  local container=$(docker ps --format "table {{.Names}}" | tail -n +2 | fzf)
  if [ -n "$container" ]; then
    docker exec -it $container /bin/bash
  fi
}

# Get all resources in namespace
kall() {
  local namespace=${1:-$(kubectl config view --minify --output 'jsonpath={..namespace}')}
  kubectl get all -n $namespace
}

# Quick helm upgrade with values
hup() {
  if [ $# -lt 2 ]; then
    echo "Usage: hup <release-name> <chart> [values-file]"
    return 1
  fi
  
  local release=$1
  local chart=$2
  local values=${3:-values.yaml}
  
  if [ -f "$values" ]; then
    helm upgrade --install $release $chart -f $values
  else
    helm upgrade --install $release $chart
  fi
}

# ============================================
# ENVIRONMENT VARIABLES
# ============================================

# History
HISTSIZE=10000
SAVEHIST=10000
setopt SHARE_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_ALL_DUPS

# Enable completions if available
if command -v kubectl >/dev/null 2>&1; then
  source <(kubectl completion zsh) 2>/dev/null
  complete -F __start_kubectl k 2>/dev/null
fi

if command -v helm >/dev/null 2>&1; then
  source <(helm completion zsh) 2>/dev/null
fi

# FZF configuration
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --border"
export FZF_DEFAULT_COMMAND="find . -type f -not -path '*/\.git/*' -not -path '*/node_modules/*' -not -path '*/bin/*' -not -path '*/obj/*'"

# Editor
export EDITOR="code"

# Path additions
export PATH="$PATH:$HOME/.local/bin"
export PATH="$PATH:$HOME/.dotnet/tools"

# Load fzf if available
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh

EOF

echo "Advanced .zshrc configuration created successfully!"
```

## Step 6: Install Essential Kubernetes Tools

```bash { name=install-k8s-tools }
# Create local bin directory if it doesn't exist
mkdir -p "$HOME/.local/bin"

# Function to detect OS
detect_os() {
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        echo "linux"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        echo "darwin"
    else
        echo "unknown"
    fi
}

OS=$(detect_os)
ARCH=$(uname -m)

# Convert architecture names
case $ARCH in
    x86_64) ARCH="amd64" ;;
    aarch64) ARCH="arm64" ;;
    arm64) ARCH="arm64" ;;
esac

echo "Detected OS: $OS, Architecture: $ARCH"

# Install kubectx and kubens
if [ ! -f "$HOME/.local/bin/kubectx" ]; then
    echo "Installing kubectx and kubens..."
    
    if [[ "$OS" == "darwin" ]] && command -v brew >/dev/null; then
        brew install kubectx
    else
        # Install from GitHub releases
        curl -sL "https://github.com/ahmetb/kubectx/releases/latest/download/kubectx_v0.9.5_${OS}_${ARCH}.tar.gz" | tar -xz -C /tmp
        curl -sL "https://github.com/ahmetb/kubectx/releases/latest/download/kubens_v0.9.5_${OS}_${ARCH}.tar.gz" | tar -xz -C /tmp
        
        mv /tmp/kubectx "$HOME/.local/bin/"
        mv /tmp/kubens "$HOME/.local/bin/"
        chmod +x "$HOME/.local/bin/kubectx"
        chmod +x "$HOME/.local/bin/kubens"
    fi
    echo "kubectx and kubens installed!"
else
    echo "kubectx and kubens already installed"
fi
```

## Step 7: Install Advanced Tools

```bash { name=install-advanced-tools }
# Install k9s (Kubernetes TUI)
if [ ! -f "$HOME/.local/bin/k9s" ]; then
    echo "Installing k9s..."
    
    if [[ "$OS" == "darwin" ]] && command -v brew >/dev/null; then
        brew install k9s
    else
        K9S_VERSION=$(curl -s https://api.github.com/repos/derailed/k9s/releases/latest | grep tag_name | cut -d '"' -f 4)
        curl -sL "https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz" | tar -xz -C /tmp
        mv /tmp/k9s "$HOME/.local/bin/"
        chmod +x "$HOME/.local/bin/k9s"
    fi
    echo "k9s installed!"
else
    echo "k9s already installed"
fi

# Install stern (multi-pod log tailing)
if [ ! -f "$HOME/.local/bin/stern" ]; then
    echo "Installing stern..."
    
    if [[ "$OS" == "darwin" ]] && command -v brew >/dev/null; then
        brew install stern
    else
        STERN_VERSION=$(curl -s https://api.github.com/repos/stern/stern/releases/latest | grep tag_name | cut -d '"' -f 4)
        curl -sL "https://github.com/stern/stern/releases/latest/download/stern_${STERN_VERSION#v}_linux_amd64.tar.gz" | tar -xz -C /tmp
        mv /tmp/stern "$HOME/.local/bin/"
        chmod +x "$HOME/.local/bin/stern"
    fi
    echo "stern installed!"
else
    echo "stern already installed"
fi

# Install dive (Docker image layer analyzer)
if [ ! -f "$HOME/.local/bin/dive" ]; then
    echo "Installing dive..."
    
    if [[ "$OS" == "darwin" ]] && command -v brew >/dev/null; then
        brew install dive
    else
        DIVE_VERSION=$(curl -s https://api.github.com/repos/wagoodman/dive/releases/latest | grep tag_name | cut -d '"' -f 4)
        curl -sL "https://github.com/wagoodman/dive/releases/latest/download/dive_${DIVE_VERSION#v}_linux_amd64.tar.gz" | tar -xz -C /tmp
        mv /tmp/dive "$HOME/.local/bin/"
        chmod +x "$HOME/.local/bin/dive"
    fi
    echo "dive installed!"
else
    echo "dive already installed"
fi

# Install lazydocker (Docker TUI)
if [ ! -f "$HOME/.local/bin/lazydocker" ]; then
    echo "Installing lazydocker..."
    
    if [[ "$OS" == "darwin" ]] && command -v brew >/dev/null; then
        brew install lazydocker
    else
        curl -sL https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh | bash
        # Move to local bin if installed in different location
        if [ -f "$HOME/go/bin/lazydocker" ]; then
            mv "$HOME/go/bin/lazydocker" "$HOME/.local/bin/"
        fi
    fi
    echo "lazydocker installed!"
else
    echo "lazydocker already installed"
fi
```

## Step 8: Set Zsh as Default Shell

```bash { name=set-default-shell }
# Set zsh as the default shell
ZSH_PATH=$(which zsh)
echo "Zsh path: $ZSH_PATH"

if [ "$SHELL" != "$ZSH_PATH" ]; then
    echo "Setting zsh as default shell..."
    
    # Add zsh to /etc/shells if not already there
    if ! grep -q "$ZSH_PATH" /etc/shells; then
        echo "$ZSH_PATH" | sudo tee -a /etc/shells
    fi
    
    # Change default shell
    chsh -s "$ZSH_PATH"
    echo "Default shell changed to zsh. Please restart your terminal or run 'exec zsh' to start using the new configuration."
else
    echo "Zsh is already the default shell"
fi
```

## Step 9: Configure Powerlevel10k (Interactive)

```bash { name=configure-p10k }
# This will launch the interactive Powerlevel10k configuration
echo "Starting Powerlevel10k configuration..."
echo "If this is your first time, the configuration wizard will start automatically when you open a new zsh session."
echo ""
echo "Recommended settings:"
echo "- Style: Rainbow"
echo "- Character Set: Unicode"
echo "- Show current time: Yes"
echo "- Prompt Style: Two lines"
echo "- Enable kubectl context: Yes"
echo "- Enable git status: Yes"
echo ""
echo "To reconfigure later, run: p10k configure"

# Start new zsh session to trigger configuration
exec zsh
```

## Step 10: Verification and Testing

```bash { name=verify-installation }
echo "=== Installation Verification ==="
echo ""

# Check zsh configuration
echo "Zsh configuration:"
echo "Default shell: $SHELL"
echo "Oh My Zsh: $([ -d "$HOME/.oh-my-zsh" ] && echo "✓ Installed" || echo "✗ Missing")"
echo "Powerlevel10k: $([ -d "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k" ] && echo "✓ Installed" || echo "✗ Missing")"
echo ""

# Check plugins
echo "Plugins:"
echo "zsh-autosuggestions: $([ -d "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions" ] && echo "✓ Installed" || echo "✗ Missing")"
echo "zsh-syntax-highlighting: $([ -d "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting" ] && echo "✓ Installed" || echo "✗ Missing")"
echo "fzf: $([ -d "$HOME/.fzf" ] && echo "✓ Installed" || echo "✗ Missing")"
echo ""

# Check tools
echo "Tools:"
echo "kubectx: $(which kubectx 2>/dev/null && echo "✓ Available" || echo "✗ Missing")"
echo "kubens: $(which kubens 2>/dev/null && echo "✓ Available" || echo "✗ Missing")"
echo "k9s: $(which k9s 2>/dev/null && echo "✓ Available" || echo "✗ Missing")"
echo "stern: $(which stern 2>/dev/null && echo "✓ Available" || echo "✗ Missing")"
echo "dive: $(which dive 2>/dev/null && echo "✓ Available" || echo "✗ Missing")"
echo "lazydocker: $(which lazydocker 2>/dev/null && echo "✓ Available" || echo "✗ Missing")"
echo ""

echo "=== Quick Command Test ==="
echo "Testing aliases (these should not show 'command not found'):"
echo "k alias: $(alias k 2>/dev/null || echo "Not set")"
echo "d alias: $(alias d 2>/dev/null || echo "Not set")"
echo "h alias: $(alias h 2>/dev/null || echo "Not set")"
```

## 📖 Complete Cheat Sheet

Here's a comprehensive reference of all commands, shortcuts, and keyboard bindings available after setup:

### 🎯 Keyboard Shortcuts & Navigation

| Shortcut | Description | Example |
|----------|-------------|---------|
| `Ctrl+R` | Fuzzy search command history | Interactive history search |
| `Ctrl+T` | Insert file/directory path | Select file with fzf |
| `Alt+C` | Change directory with fuzzy finder | Navigate directories interactively |
| `Tab` | Smart auto-completion | `k get p<Tab>` → `k get pods` |
| `Ctrl+A` | Move cursor to beginning of line | Quick line navigation |
| `Ctrl+E` | Move cursor to end of line | Quick line navigation |
| `Ctrl+W` | Delete word before cursor | Quick editing |
| `Ctrl+U` | Delete entire line | Clear current command |
| `Ctrl+L` | Clear screen | `clear` command equivalent |
| `!!` | Repeat last command | `sudo !!` |
| `!<text>` | Run last command starting with text | `!k` runs last kubectl command |

### ☸️ Kubernetes Commands & Aliases

#### Basic Operations

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `k` | `kubectl` | Main kubectl shortcut | `k get pods` |
| `kgp` | `kubectl get pods` | List pods | `kgp -w` |
| `kgs` | `kubectl get svc` | List services | `kgs -A` |
| `kgd` | `kubectl get deployment` | List deployments | `kgd -o wide` |
| `kgn` | `kubectl get nodes` | List nodes | `kgn -o wide` |
| `kga` | `kubectl get all` | Get all resources | `kga -n kube-system` |
| `kd` | `kubectl describe` | Describe resource | `kd pod my-pod` |
| `kdel` | `kubectl delete` | Delete resource | `kdel pod my-pod` |
| `kl` | `kubectl logs` | Get logs | `kl -f my-pod` |
| `kex` | `kubectl exec -it` | Execute in pod | `kex my-pod -- bash` |
| `kpf` | `kubectl port-forward` | Port forward | `kpf pod/my-pod 8080:80` |

#### Advanced Aliases

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `kgpa` | `kubectl get pods --all-namespaces` | List all pods | `kgpa \| grep Running` |
| `kgsca` | `kubectl get secret --all-namespaces` | List all secrets | `kgsca \| grep tls` |
| `kgcma` | `kubectl get configmap --all-namespaces` | List all configmaps | `kgcma \| grep app` |
| `kwp` | `watch kubectl get pods` | Watch pods | `kwp` |
| `kws` | `watch kubectl get svc` | Watch services | `kws` |
| `kwd` | `watch kubectl get deployment` | Watch deployments | `kwd` |

#### Context & Namespace Management

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `kctx` | `kubectl config current-context` | Show current context | `kctx` |
| `kns` | `kubectl config view --minify --output 'jsonpath={..namespace}'` | Show current namespace | `kns` |
| `kcc` | `kubectl config use-context` | Switch context | `kcc production` |
| `kcn` | `kubectl config set-context --current --namespace` | Switch namespace | `kcn kube-system` |
| `kubectx` | External tool | List/switch contexts | `kubectx staging` |
| `kubens` | External tool | List/switch namespaces | `kubens default` |

#### Custom Functions

| Function | Description | Usage | Example |
|----------|-------------|-------|---------|
| `kswitch` | Context switcher with list | `kswitch [context]` | `kswitch production` |
| `klogs` | Fuzzy search pods → tail logs | `klogs` | Interactive pod selection |
| `kexec` | Fuzzy search pods → exec shell | `kexec` | Interactive pod selection |
| `kport` | Fuzzy search pods → port forward | `kport` | Interactive port forwarding |
| `kall` | Get all resources in namespace | `kall [namespace]` | `kall kube-system` |

### 🐳 Docker Commands & Aliases

#### Basic Docker Operations

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `d` | `docker` | Main docker shortcut | `d ps` |
| `dps` | `docker ps` | List running containers | `dps -a` |
| `dpa` | `docker ps -a` | List all containers | `dpa` |
| `di` | `docker images` | List images | `di` |
| `dex` | `docker exec -it` | Execute in container | `dex my-container bash` |
| `dl` | `docker logs` | Container logs | `dl -f my-container` |
| `dsp` | `docker system prune` | Clean up system | `dsp` |
| `dspa` | `docker system prune -a` | Clean up everything | `dspa` |
| `dvp` | `docker volume prune` | Clean up volumes | `dvp` |
| `dnp` | `docker network prune` | Clean up networks | `dnp` |

#### Docker Compose

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `dc` | `docker-compose` | Docker compose shortcut | `dc up -d` |
| `dcu` | `docker-compose up` | Start services | `dcu -d` |
| `dcd` | `docker-compose down` | Stop services | `dcd --volumes` |
| `dcb` | `docker-compose build` | Build services | `dcb --no-cache` |
| `dcl` | `docker-compose logs` | View logs | `dcl -f web` |
| `dcps` | `docker-compose ps` | List services | `dcps` |

#### Custom Functions

| Function | Description | Usage | Example |
|----------|-------------|-------|---------|
| `dshell` | Fuzzy search containers → get shell | `dshell` | Interactive container selection |

### ⚓ Helm Commands & Aliases

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `h` | `helm` | Main helm shortcut | `h list` |
| `hls` | `helm list` | List releases | `hls -A` |
| `hla` | `helm list --all-namespaces` | List all releases | `hla` |
| `hi` | `helm install` | Install chart | `hi my-app ./chart` |
| `hu` | `helm upgrade` | Upgrade release | `hu my-app ./chart` |
| `hd` | `helm delete` | Delete release | `hd my-app` |
| `hs` | `helm status` | Release status | `hs my-app` |
| `hr` | `helm repo` | Manage repositories | `hr add stable https://...` |

#### Custom Functions

| Function | Description | Usage | Example |
|----------|-------------|-------|---------|
| `hup` | Quick upgrade with values | `hup <release> <chart> [values]` | `hup my-app ./chart values.yaml` |

### ☁️ Cloud Provider Commands

#### AWS CLI

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `awsp` | `aws sts get-caller-identity` | Show current AWS profile | `awsp` |
| `awsr` | `aws configure list` | Show AWS configuration | `awsr` |

#### Azure CLI

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `azl` | `az login` | Login to Azure | `azl` |
| `azs` | `az account show` | Show current subscription | `azs` |
| `azg` | `az group list` | List resource groups | `azg` |

#### Google Cloud CLI

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `gcpl` | `gcloud auth list` | List authenticated accounts | `gcpl` |
| `gcpp` | `gcloud config list project` | Show current project | `gcpp` |
| `gcps` | `gcloud config set project` | Set current project | `gcps my-project` |

### 🔧 .NET Development Commands

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `dn` | `dotnet` | Main dotnet shortcut | `dn --version` |
| `dnb` | `dotnet build` | Build project | `dnb --configuration Release` |
| `dnr` | `dotnet run` | Run project | `dnr --project ./src/App` |
| `dnt` | `dotnet test` | Run tests | `dnt --logger console` |
| `dnp` | `dotnet publish` | Publish project | `dnp -c Release` |
| `dnw` | `dotnet watch` | Watch and restart | `dnw run` |

### 🧭 Navigation & File Operations

| Alias | Full Command | Description | Example |
|-------|--------------|-------------|---------|
| `ll` | `ls -la` | List detailed | `ll` |
| `la` | `ls -la` | List all detailed | `la` |
| `..` | `cd ..` | Go up one directory | `..` |
| `...` | `cd ../..` | Go up two directories | `...` |
| `....` | `cd ../../..` | Go up three directories | `....` |
| `z` | Smart directory jump | Jump to frequently used dirs | `z project` |

### 🔍 Advanced Tools & TUIs

| Tool | Description | Usage | Key Features |
|------|-------------|-------|-------------|
| `k9s` | Kubernetes TUI | `k9s` | Visual cluster management |
| `lazydocker` | Docker TUI | `lazydocker` | Visual container management |
| `stern` | Multi-pod logs | `stern <selector>` | `stern app=frontend` |
| `dive` | Image analysis | `dive <image>` | `dive nginx:latest` |
| `fzf` | Fuzzy finder | Built into many functions | Interactive file/command selection |

### 🎮 K9s Keyboard Shortcuts (Inside K9s)

| Key | Action | Description |
|-----|--------|-------------|
| `:pods` | Navigate to pods | Switch to pods view |
| `:svc` | Navigate to services | Switch to services view |
| `:deploy` | Navigate to deployments | Switch to deployments view |
| `:ns` | Navigate to namespaces | Switch to namespaces view |
| `Ctrl+A` | Show all namespaces | Toggle all namespaces view |
| `d` | Describe resource | Show resource details |
| `l` | View logs | Show pod logs |
| `s` | Shell into pod | Execute shell |
| `y` | YAML view | Show resource YAML |
| `e` | Edit resource | Edit resource inline |
| `Ctrl+D` | Delete resource | Delete selected resource |
| `/` | Filter resources | Search/filter current view |
| `Esc` | Clear filter | Remove current filter |
| `:q` | Quit | Exit k9s |

### 🔄 Git Integration (Oh My Zsh Git Plugin)

| Alias | Full Command | Description |
|-------|--------------|-------------|
| `g` | `git` | Git shortcut |
| `ga` | `git add` | Add files |
| `gaa` | `git add --all` | Add all files |
| `gc` | `git commit -v` | Commit with verbose |
| `gcm` | `git commit -m` | Commit with message |
| `gco` | `git checkout` | Checkout branch |
| `gb` | `git branch` | List branches |
| `gp` | `git push` | Push to remote |
| `gl` | `git pull` | Pull from remote |
| `gst` | `git status` | Show status |
| `glog` | `git log --oneline --decorate --graph` | Pretty log |

### 🎨 Powerlevel10k Prompt Segments

Your prompt will show contextual information:

- **Current directory** (shortened intelligently)
- **Git branch and status** (if in git repo)
- **Kubectl context and namespace** (if kubectl available)
- **Exit code** (if last command failed)
- **Background jobs** (if any running)
- **Execution time** (for long-running commands)
- **AWS/Azure/GCP context** (if configured)

### 💡 Pro Tips & Power User Features

1. **Fuzzy History Search**: `Ctrl+R` then type partial command
2. **Directory Jumping**: Use `z partial-name` to jump to frequently used directories
3. **Tab Completion**: Works for kubectl resources, docker containers, helm releases
4. **Command Substitution**: Use `$(kgp -o name)` in other commands
5. **Pipe to fzf**: `kgp | fzf` for interactive selection
6. **Watch Commands**: Prefix with `watch` for live updates: `watch kgp`
7. **Multiple Context Work**: Use `kubectx` and `kubens` for quick switching
8. **Log Following**: Most log commands support `-f` flag for following

### 🚨 Emergency Commands

| Command | Description | Use Case |
|---------|-------------|----------|
| `k get events --sort-by=.metadata.creationTimestamp` | Recent cluster events | Troubleshooting |
| `k top nodes` | Node resource usage | Performance issues |
| `k top pods` | Pod resource usage | Resource debugging |
| `dsp -f` | Force clean Docker | Space issues |
| `k describe node <node>` | Node details | Node problems |
| `stern . --all-namespaces` | All pod logs | System-wide issues |

## Usage Examples and Quick Start

Now that your zsh is configured, here are some quick examples to get you started:

```bash { name=usage-examples }
echo "=== Usage Examples ==="
echo ""
echo "🎯 Quick Start Commands:"
echo "  k9s                          # Launch Kubernetes dashboard"
echo "  lazydocker                   # Launch Docker dashboard"
echo "  kubectx                      # List and switch contexts"
echo "  kubens                       # List and switch namespaces"
echo ""
echo "☸️ Kubernetes shortcuts:"
echo "  k get pods                   # List pods"
echo "  kgp -w                       # Watch pods"
echo "  klogs                        # Fuzzy search pods → tail logs"
echo "  kexec                        # Fuzzy search pods → exec shell"
echo "  kall production              # Show all resources in production namespace"
echo ""
echo "🐳 Docker shortcuts:"
echo "  d ps                         # List containers"
echo "  dshell                       # Fuzzy search containers → get shell"
echo "  dc up -d                     # Start compose services in background"
echo "  dive nginx:latest            # Analyze image layers"
echo ""
echo "🧭 Navigation & Search:"
echo "  z myproject                  # Jump to project directory"
echo "  Ctrl+R                       # Fuzzy search command history"
echo "  Ctrl+T                       # Insert file path with fuzzy finder"
echo "  ll                           # Detailed file listing"
echo ""
echo "🔧 .NET Development:"
echo "  dnw run                      # dotnet watch run"
echo "  dnt --logger console         # dotnet test with output"
echo "  dnb --configuration Release  # Build release version"
echo ""
echo "📊 Monitoring & Debugging:"
echo "  stern app=frontend           # Tail logs from all frontend pods"
echo "  k top nodes                  # Show node resource usage"
echo "  k get events --sort-by=.metadata.creationTimestamp  # Recent events"
```

## Final Steps

```bash { name=final-steps }
echo "=== Setup Complete! ==="
echo ""
echo "🎉 Your zsh environment is now configured for cloud-native development!"
echo ""
echo "Next steps:"
echo "1. Restart your terminal or run 'exec zsh'"
echo "2. If Powerlevel10k configuration doesn't start automatically, run 'p10k configure'"
echo "3. Try some of the aliases and functions listed in the usage examples"
echo "4. Customize further by editing ~/.zshrc as needed"
echo ""
echo "Pro tips:"
echo "- Use 'z' to jump to frequently used directories"
echo "- Use Ctrl+R for fuzzy command history search"
echo "- Use Tab completion extensively - it's very smart!"
echo "- Check out k9s for a visual Kubernetes interface"
echo ""
echo "Your old .zshrc has been backed up with a timestamp."
echo "Configuration files created in: $HOME/.zshrc"
echo "Tools installed in: $HOME/.local/bin/"
```

## Troubleshooting

If you encounter issues:

```bash { name=troubleshooting }
echo "=== Troubleshooting ==="
echo ""
echo "Common issues and solutions:"
echo ""
echo "1. 'command not found' errors:"
echo "   - Restart your terminal or run 'exec zsh'"
echo "   - Check if ~/.local/bin is in your PATH: echo \$PATH"
echo ""
echo "2. Kubectl completions not working:"
echo "   - Make sure kubectl is installed and in PATH"
echo "   - Run 'source <(kubectl completion zsh)' manually"
echo ""
echo "3. Powerlevel10k not showing correctly:"
echo "   - Install a Nerd Font (recommended: MesloLGS NF)"
echo "   - Configure your terminal to use the font"
echo "   - Run 'p10k configure' to reconfigure"
echo ""
echo "4. fzf not working:"
echo "   - Run '~/.fzf/install' to reinstall"
echo "   - Source the configuration: 'source ~/.fzf.zsh'"
echo ""
echo "5. Tools not found:"
echo "   - Check installation: ls -la ~/.local/bin/"
echo "   - Add to PATH: export PATH=\"\$PATH:\$HOME/.local/bin\""
echo ""
echo "For more help, check the official documentation:"
echo "- Oh My Zsh: https://ohmyz.sh/"
echo "- Powerlevel10k: https://github.com/romkatv/powerlevel10k"
echo "- fzf: https://github.com/junegunn/fzf"
```