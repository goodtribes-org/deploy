# Getting Started

Everything you need to set up a working local development environment and start contributing to the Goodtribes platform.

---

## 1. Accounts

### GitHub
- Create a [GitHub account](https://github.com) if you don't have one.
- Ask an org admin to invite you to the **goodtribes-org** organisation.
- Enable two-factor authentication (required by the org).

### Claude (Anthropic)
The AI-assisted workflow runs entirely through [Claude Code](https://claude.ai/code), Anthropic's AI coding CLI.

- Create an account at [claude.ai](https://claude.ai).
- Subscribe to **Claude Pro** (individual) or ensure the org has a **Claude Teams** plan. The background worker agents require a paid plan.
- You also need a personal **Anthropic API key** for the CLI: visit [console.anthropic.com](https://console.anthropic.com) → API keys → Create key.

---

## 2. Install required tools

### Core tools

```bash
# Git
# macOS
brew install git
# Ubuntu/Debian
sudo apt install git

# GitHub CLI (gh) — used by all workflow agents
# macOS
brew install gh
# Ubuntu/Debian
sudo apt install gh
# or: https://github.com/cli/cli/releases

# Claude Code CLI — the AI coding tool
npm install -g @anthropic-ai/claude-code
```

### Authenticate

```bash
# Log in to GitHub CLI — follow the interactive prompts
gh auth login

# Set your Anthropic API key for Claude Code
export ANTHROPIC_API_KEY=sk-ant-...
# Add to your shell profile to persist:
echo 'export ANTHROPIC_API_KEY=sk-ant-...' >> ~/.bashrc   # or ~/.zshrc
```

### Development tools

```bash
# Node.js 20+ and npm (required for kickfix and asylguiden.se)
# macOS
brew install node
# Ubuntu/Debian — use nvm for version management:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install 20 && nvm use 20

# Docker and Docker Compose (required for local dev)
# https://docs.docker.com/engine/install/
```

### Cluster tools (optional — only needed for infrastructure work)

```bash
# kubectl
# macOS
brew install kubectl
# Ubuntu/Debian
sudo apt install kubectl

# Helm
# macOS
brew install helm
# Ubuntu/Debian
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## 3. Folder structure

All repositories should live under a common `~/projects/` folder. The monorepo (`goodtribes.org`) contains the source code for all three projects as subdirectories.

```
~/projects/
├── goodtribes.org/        # goodtribes-org/goodtribes.org  ← main monorepo
│   ├── kickfix/           # kickfix source code
│   ├── asylguiden.se/     # asylguiden.se source code
│   ├── goodtribes.org/    # future project placeholder
│   ├── argocd/            # deploy key and pull secret scripts
│   └── .claude/
│       └── commands/      # /gh-start, /gh-request, /gh-plan, /ticket, ...
├── deploy/                # goodtribes-org/deploy  ← this repo
└── agent/                 # goodtribes-org/agent   ← shared Claude skills
```

---

## 4. Clone all repositories

```bash
mkdir -p ~/projects && cd ~/projects

# Main monorepo (source code for all projects)
git clone git@github.com:goodtribes-org/goodtribes.org.git

# GitOps manifest repo (this repo)
git clone git@github.com:goodtribes-org/deploy.git

# Shared Claude Code skills
git clone git@github.com:goodtribes-org/agent.git
```

> **SSH keys** — if you haven't set up SSH for GitHub yet:
> ```bash
> ssh-keygen -t ed25519 -C "your@email.com"
> cat ~/.ssh/id_ed25519.pub  # paste this into GitHub → Settings → SSH keys
> ```

---

## 5. Install Claude Code skills

Copy the shared agent skills into your Claude config so the `/gh-start`, `/gh-plan`, and `/ticket` commands are available:

```bash
cp -r ~/projects/agent/.claude/commands/. ~/.claude/commands/
```

> The monorepo also ships its own `.claude/commands/` — these are picked up automatically when you open Claude Code from inside `~/projects/goodtribes.org/`.

---

## 6. Cluster access (optional)

If you need to inspect or manage the Kubernetes cluster:

```bash
# Place the kubeconfig provided by the cluster admin at:
~/.kube/confighrb

# Test access:
kubectl --kubeconfig ~/.kube/confighrb get nodes
```

The cluster has three namespaces: `kickfix`, `asylguiden`, `goodtribes`.

---

## 7. Local development

Each project can be run locally with Docker Compose:

```bash
# kickfix (frontend :3003, backend API :5000)
cd ~/projects/goodtribes.org/kickfix
docker compose up --build

# asylguiden.se (frontend :3000, Strapi CMS :1337)
cd ~/projects/goodtribes.org/asylguiden.se
docker compose up --build
```

See the root `CLAUDE.md` in the monorepo for full environment variable requirements.

---

## Next steps

Read the [development workflow guide](workflow.md) to understand how feature requests move through the AI-assisted planning pipeline.
