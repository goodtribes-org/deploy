# goodtribes-org/deploy

GitOps manifest repo. Rendered Kubernetes manifests are committed here by CI and picked up by the cluster's ArgoCD operator.

| Directory | Project |
|-----------|---------|
| `asylguiden/` | asylguiden.se |
| `kickfix/` | kickfix.se |
| `goodtribes/` | goodtribes.org |

---

## Platform overview

Three independent web applications under the **goodtribes-org** GitHub organisation, developed with an AI-assisted planning workflow powered by [Claude Code](https://claude.ai/code).

| Repo | Description | Stack |
|------|-------------|-------|
| [goodtribes-org/goodtribes.org](https://github.com/goodtribes-org/goodtribes.org) | Main monorepo — source code for all three projects | React / Next.js / Express / Strapi |
| [goodtribes-org/kickfix](https://github.com/goodtribes-org/kickfix) | Swedish freelance job marketplace | React 19 + Express + MongoDB |
| [goodtribes-org/asylguiden.se](https://github.com/goodtribes-org/asylguiden.se) | Swedish refugee resource site | Next.js 16 + Strapi 5 + PostgreSQL |
| [goodtribes-org/agent](https://github.com/goodtribes-org/agent) | Shared Claude Code skills for the team | — |
| [goodtribes-org/deploy](https://github.com/goodtribes-org/deploy) | This repo — Kubernetes manifests (GitOps) | Helm / ArgoCD |

## Project boards

| Board | Link |
|-------|------|
| goodtribes.org | https://github.com/orgs/goodtribes-org/projects/2 |
| kickfix | https://github.com/orgs/goodtribes-org/projects/3 |
| asylguiden.se | https://github.com/orgs/goodtribes-org/projects/4 |

## Documentation

- [Getting started — accounts, tools, and repo setup](docs/setup.md)
- [Development workflow — the AI-assisted planning pipeline](docs/workflow.md)
