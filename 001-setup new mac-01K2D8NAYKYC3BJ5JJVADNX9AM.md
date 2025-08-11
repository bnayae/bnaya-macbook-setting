---
runme:
  document:
    relativePath: 001-setup new mac.md
  session:
    id: 01K2D8NAYKYC3BJ5JJVADNX9AM
    updated: 2025-08-11 22:14:58+03:00
---

# Setup

## Brew

Subline-Text

```bash
brew install --cask sublime-text
```

And follow the instructions

## Docker

```bash
brew install docker
```

## Create shortcuts

Terminal

// TODO

## KeePass

```bash
brew install keepassxc
```

# VS Code

```bash
brew install --cask visual-studio-code
```

Upgrade

```bash
brew upgrade --cask visual-studio-code
```

- [Open from Finder](./VsCode/OpenInFinder.md)

### Quick Open Vs Code

From Terminal

```bash
sudo ln -sf "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code" /usr/local/bin/code
```

## AI

-   Ollama:

```bash
brew install ollama
```

- Open Web UI

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

- Ch*******ll

```bash
brew search ch*******ll
```

- LM Studio (LLM desktop)

```bash
brew install --cask lm-studio
```

- Tranformers.js (run model in the browser)

```bash
npm init -y
npm install @huggingface/transformers
```

## Node

- Install Node.js

```bash
brew install node
```

Check it

```bash
node -v
npm -v
```

- Install Yarn

```bash
brew install yarn
```

Check it

```bash
yarn -v
```


