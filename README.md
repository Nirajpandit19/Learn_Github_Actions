# Learn GitHub Actions

> A hands-on learning repository covering GitHub Actions from basics to real-world deployments.
> Each topic has its own dedicated notes file with explanations, workflow examples, and key concepts.

---

## Repository Structure

```
LEARN_GITHUB_ACTIONS/
│
├── .github/
│   └── workflows/
│       ├── hello.yaml                          # Hello World workflow
│       ├── variables_and_secrets.yaml          # Variables and secrets demo
│       ├── actions.yaml                        # Actions (uses:) demo
│       ├── inputs.yaml                         # Inputs demo
|       ├── outputs.yaml                        # Outputs demo 
│       └── deploy_static_website_on_s3.yaml    # S3 + CloudFront deployment
│
├── docs/
│   ├── 01_basics_github_actions.md             # Workflows, jobs, steps, triggers
│   ├── 02_variables_and_secrets.md             # Variables, secrets, default env vars
│   ├── 03_actions.md                           # uses: keyword, marketplace actions
│   ├── 04_inputs_outputs.md                    # Inputs/parameters and outputs
│   └── 05_deploy_to_s3_cloudfront.md           # Project — static site deployment
│
├── projects/
│   └── static-site/
│       └── index.html                          # Deployed via GitHub Actions to S3
│
└── README.md                                   # This file — repo index
```

---

## Topics Covered

| # | Topic | Notes | Workflow |
|---|-------|-------|----------|
| 01 | Basics — Workflows, Jobs, Steps, Triggers | [docs/01_basics_github_actions.md](docs/01_basics_github_actions.md) | [hello.yaml](.github/workflows/hello.yaml) |
| 02 | Variables and Secrets | [docs/02_variables_and_secrets.md](docs/02_variables_and_secrets.md) | [variables_and_secrets.yaml](.github/workflows/variables_and_secrets.yaml) |
| 03 | Actions — `uses:` keyword | [docs/03_actions.md](docs/03_actions.md) | [actions.yaml](.github/workflows/actions.yaml) |
| 04 | Inputs / Parameters and Outputs | [docs/04_inputs_outputs.md](docs/04_inputs_outputs.md) | [inputs_outputs.yaml](.github/workflows/inputs_outputs.yaml) |

---

## Projects

| # | Project | Tech Stack | Docs | Live |
|---|---------|-----------|------|------|
| 01 | Deploy Static Site to S3 + CloudFront | GitHub Actions, AWS S3, CloudFront, OAC, IAM | [docs/05_deploy_to_s3_cloudfront.md](docs/05_deploy_to_s3_cloudfront.md) | [View Site](https://djo3z7qjj9kcz.cloudfront.net) |

---

## Key Concepts at a Glance

### Workflow Structure
```
Workflow (.github/workflows/name.yaml)
└── Jobs (independent units, run in parallel by default)
    └── Steps (individual commands or actions within a job)
```

### Trigger Quick Reference
| Trigger | When it fires |
|---------|--------------|
| `push` | On every push to specified branch |
| `pull_request` | When a PR is opened or updated |
| `workflow_dispatch` | Manually from GitHub UI |
| `schedule` | On a cron schedule |
| `workflow_call` | Called by another workflow |

### vars. vs secrets.
| Use | Keyword | Example |
|-----|---------|---------|
| Non-sensitive dynamic value | `vars.` | `${{ vars.AWS_REGION }}` |
| Sensitive value | `secrets.` | `${{ secrets.API_KEY }}` |

### Output Scopes
| Scope | Syntax | When to use |
|-------|--------|-------------|
| Same job | `${{ steps.STEP_ID.outputs.KEY }}` | Sharing between steps |
| Different job | `${{ needs.JOB_NAME.outputs.KEY }}` | Sharing between jobs |

---

## CI vs CD — Trigger Strategy

```
pull_request → main     Use for CI  (build, test, security scan — on proposed code)
push → main             Use for CD  (deploy — only after PR is merged and approved)
```

---

## Learning Progress

- [x] Workflows, Jobs, Steps
- [x] Triggers — push, pull_request, workflow_dispatch, branch-specific
- [x] Variables and Secrets
- [x] Actions — `uses:` keyword, marketplace actions
- [x] Inputs / Parameters and Outputs
- [x] Project — Deploy Static Site to S3 + CloudFront
- [ ] Reusable Workflows
- [ ] Matrix Strategy
- [ ] Artifacts and Caching
- [ ] Self-hosted Runners
- [ ] Full CI/CD Pipeline — Java + Docker + Kubernetes

---

*Notes by Pandit Niraj Raj | [GitHub](https://github.com/Nirajpandit19) | [LinkedIn](https://linkedin.com/in/pandit-niraj-raj)*
