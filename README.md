# goodtribes-org/deploy

GitOps manifest repo. Rendered Kubernetes manifests are committed here by CI and picked up by the cluster's ArgoCD operator.

| Directory | Project |
|-----------|---------|
| `asylguiden/` | asylguiden.se |
| `kickfix/` | kickfix.se |
| `goodtribes/` | goodtribes.org |

---

## Starting the agent workflow

The AI planning pipeline runs from the monorepo. Open [Claude Code](https://claude.ai/code) inside `~/projects/goodtribes.org/` and run one command:

```
/gh-start
```

This launches two background workers that watch all three project boards simultaneously:

| Worker | Watches | Does |
|--------|---------|------|
| `gh-request` | issues in `new` | Validates scope, posts an outline plan, moves card to `request` |
| `gh-plan` | issues in `plan` | Reads the codebase, writes a file-level implementation plan, moves card to `review` |

Both workers poll every 5 minutes. They run until you stop them — one `/gh-start` covers all three projects.

---

## Feeding the pipeline

1. **File a feature request** in the relevant repo using the Feature Request template:
   - kickfix feature → [goodtribes-org/kickfix/issues/new](https://github.com/goodtribes-org/kickfix/issues/new/choose)
   - asylguiden.se feature → [goodtribes-org/asylguiden.se/issues/new](https://github.com/goodtribes-org/asylguiden.se/issues/new/choose)
   - Cross-project or unsure → [goodtribes-org/deploy/issues/new](https://github.com/goodtribes-org/deploy/issues/new/choose)

2. **Add the issue to the correct project board** and set its status to **`new`**.

3. **The workers take it from there** — within 5 minutes `gh-request` will post an outline on the issue and move the card to `request`.

---

## Human checkpoints

The workers never move a card past these two points without human approval:

| Transition | What to check |
|------------|---------------|
| `request` → `plan` | Review the outline comment. Scope, steps, and files look right? Move the card. |
| `review` → `apply` | Review the implementation plan comment. Correct and safe to implement? Move the card. |

Once a card reaches `apply`, run `/ticket` in Claude Code to implement the plan, or do it manually.

---

## Pipeline at a glance

```
new  →  request  →  plan  →  review  →  apply  →  test
 ↑          ↑                    ↑
auto     human             human
(gh-request  approves)     approves
 posts       outline         plan
 outline)
```

---

## Project boards

| Project | Board |
|---------|-------|
| goodtribes.org | https://github.com/orgs/goodtribes-org/projects/2 |
| kickfix | https://github.com/orgs/goodtribes-org/projects/3 |
| asylguiden.se | https://github.com/orgs/goodtribes-org/projects/4 |

---

## Documentation

- [Getting started — accounts, tools, and repo setup](docs/setup.md)
- [Full workflow reference](docs/workflow.md)
