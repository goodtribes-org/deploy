# Development Workflow

The Goodtribes platform uses an AI-assisted planning pipeline built on top of [Claude Code](https://claude.ai/code) and GitHub Projects. Human decisions gate each phase — the AI handles research and writing, humans approve before work begins.

---

## Pipeline overview

```
[Issue filed]
      ↓
    new         ← issue lands here; workers pick it up
      ↓  (auto — gh-request worker)
  request       ← outline posted as comment; human reviews
      ↓  (human moves card)
    plan        ← detailed implementation plan requested
      ↓  (auto — gh-plan worker)
   review       ← full plan posted; human approves
      ↓  (human moves card)
    apply       ← implementation begins (/ticket or manual)
      ↓
    test        ← QA and verification
      ↓
  [PR merged → CI/CD → cluster]
```

**Two humans decisions gate the pipeline**: moving a card from `request` → `plan` (approve the outline) and from `review` → `apply` (approve the implementation plan). Everything else is automated.

---

## Project boards

Each project has its own GitHub Project board:

| Board | URL |
|-------|-----|
| goodtribes.org | https://github.com/orgs/goodtribes-org/projects/2 |
| kickfix | https://github.com/orgs/goodtribes-org/projects/3 |
| asylguiden.se | https://github.com/orgs/goodtribes-org/projects/4 |

---

## Step 1 — File a feature request

Open an issue in the repo that matches the feature:
- **kickfix feature** → issue in [goodtribes-org/kickfix](https://github.com/goodtribes-org/kickfix/issues/new/choose)
- **asylguiden.se feature** → issue in [goodtribes-org/asylguiden.se](https://github.com/goodtribes-org/asylguiden.se/issues/new/choose)
- **Cross-project or unsure** → issue in [goodtribes-org/deploy](https://github.com/goodtribes-org/deploy/issues/new/choose)

All repos have a **Feature Request** template — use it. The template collects the problem, proposed solution, priority, and acceptance criteria. It also auto-applies the `feature-request` label.

After creating the issue, **add it to the correct project board** and set its status to **`new`**. The workers will pick it up from there.

---

## Step 2 — Start the workers

From inside the monorepo, open Claude Code and run:

```
/gh-start
```

This launches two background agents that poll all three project boards every 5 minutes:

| Worker | Watches | Does |
|--------|---------|------|
| `gh-request` | `new` status | Reads the issue, checks scope/stack/sensitive data, posts an outline plan, moves card to `request` |
| `gh-plan` | `plan` status | Reads the codebase, writes a file-level implementation plan, posts it as a comment, moves card to `review` |

Workers run indefinitely until you stop them. You only need to run `/gh-start` once per session — both workers handle all three boards.

---

## Stage: `new` → `request` (gh-request worker)

The `gh-request` worker processes each `new` issue automatically:

1. **Identifies the sub-project** from the issue title/body/labels (`kickfix`, `asylguiden.se`, or `goodtribes.org`).
2. **Scope check** — flags if the request is too large (>10 files, multiple phases, 3+ new components). Large issues are returned to `new` with a comment asking for a breakdown.
3. **Sensitive data check** — flags PII, payment card data, health data, government IDs. These are warnings, not blockers, but require explicit acknowledgement.
4. **Stack check** — warns if the request would introduce a new database, cache, queue, or significant external API not already in the project.
5. **Posts an outline** as a GitHub comment:
   - Scope estimate (files affected)
   - Sensitive data and stack flags
   - Ordered implementation steps (file-level)
   - Testing instructions

After posting, the card moves to **`request`**. The worker does not move it further — that is your job.

### Human action: review the outline

Read the outline comment on the issue. If it looks right, **move the card from `request` to `plan`** on the project board. If something is wrong, edit the issue and leave the card at `request` — the worker will re-evaluate on the next loop (or you can manually re-run `/gh-request`).

---

## Stage: `plan` → `review` (gh-plan worker)

The `gh-plan` worker processes each `plan` issue automatically:

1. **Reads the codebase** — opens the relevant source files using the outline from `gh-request` as a guide. Uses `ls` to confirm paths before referencing them.
2. **Writes a detailed implementation plan**:
   - Background (what the issue asks for and why)
   - Step-by-step file changes in dependency order
   - Code notes (naming conventions, patterns to follow, things to avoid)
   - Verification steps (exact commands, expected output, edge cases)
3. **Posts the plan** as a GitHub comment.
4. Card moves to **`review`**.

### Human action: approve the plan

Read the implementation plan. This is the last checkpoint before code is written. If the plan is correct, **move the card from `review` to `apply`**. If something needs to change, edit the issue or comment your correction and move the card back to `plan` — the worker will rewrite the plan.

---

## Stage: `apply` — implementation

Once a card is in **`apply`**, implementation begins. There are two options:

### Option A — AI implementation with `/ticket`

Open Claude Code from the monorepo root and run:

```
/ticket
```

The `/ticket` worker reads the implementation plan from the issue comments and executes it — creating a branch, making file changes, running tests, and opening a PR.

### Option B — Manual implementation

Implement the plan yourself, following the steps in the implementation plan comment. Open a PR against the project's main branch when done.

---

## Stage: `test` — QA

Move the card to **`test`** when the PR is open. Run the verification steps from the implementation plan. When all checks pass, merge the PR.

---

## CI/CD — from merge to cluster

Merging a PR to `main` in any source repo triggers a GitHub Actions pipeline:

1. Docker image built and pushed to `ghcr.io/goodtribes-org/<project>:<sha>`
2. Helm chart rendered with the new image tag
3. Rendered manifests committed to this repo (`goodtribes-org/deploy`)
4. ArgoCD detects the change and syncs the cluster

Deployments typically take 2–3 minutes after merge. Monitor with:

```bash
kubectl --kubeconfig ~/.kube/confighrb get pods -n kickfix
kubectl --kubeconfig ~/.kube/confighrb get pods -n asylguiden
```

---

## Quick reference

| Command | Where | What it does |
|---------|-------|--------------|
| `/gh-start` | Claude Code, monorepo root | Launch both AI workers (request + plan) |
| `/gh-request` | Claude Code, monorepo root | Run the request-outline worker once |
| `/gh-plan` | Claude Code, monorepo root | Run the plan-writing worker once |
| `/ticket` | Claude Code, monorepo root | Implement the approved plan for one issue |
| `gh project list --owner goodtribes-org` | terminal | List all project boards |

---

## Tips

- **Workers are stateless** — if a worker crashes, just run `/gh-start` again. It will resume from the current board state.
- **Move cards yourself** — the workers never move cards from `request` → `plan` or `review` → `apply`. Those transitions require a human.
- **One issue at a time** — each worker processes one issue per iteration, then loops back. Issues are processed in board order.
- **Labels are applied automatically** — `gh-request` applies the sub-project label (`kickfix`, `asylguiden.se`, `goodtribes.org`) and stage labels (`request`, `plan`, `review`) as the issue moves through the pipeline.
