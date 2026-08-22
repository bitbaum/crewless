# crewless

**One person. 36 repositories. No crew.**

This is the harness that lets a solo builder run a software company: AI agents write most of the code, and the machinery in this repo is what makes that safe — nothing merges by hand, nothing deploys unverified, nothing publishes with a credential that could leak.

It is not a framework and it was not designed. Every script here was **extracted from a live fleet** of 36 production repos ([orangecat.ch](https://orangecat.ch), [fleetcrown.orangecat.ch](https://fleetcrown.orangecat.ch), [datacat.orangecat.ch](https://datacat.orangecat.ch), and 30+ more) after the third time the same problem was solved twice. The comments carry the outage reports that shaped them.

## The three ideas

1. **Nobody merges by hand.** A green, non-draft PR is the decision to ship. A sweep merges it, re-arms CI (a `GITHUB_TOKEN` push triggers no workflows — the number-one silent failure in GitHub Actions automation), and reconciles the deploy.
2. **Deployment and publishing are reconcilers, not triggers.** Don't fire-and-forget on an event that might not come. Ask "does the world match the repo?" on a timer, and repair it if not. A missed deploy or an unpublished version heals within the hour, unnoticed.
3. **One definition of verified.** Each repo exposes a single `verify` script (lint + typecheck + build + test) that CI calls verbatim. Green locally ⇒ green in CI. A gate that can't go red — `continue-on-error`, `--if-present`, a test glob matching zero files — is worse than no gate: it's a ✓ vouching for broken code.

## What's inside

| Path | What it is | Battle scars in the comments |
|------|-----------|------------------------------|
| `sweep/auto-merge-sweep.sh` | Merge every ready+green PR, re-arm CI/CD, reconcile deploys | Drifted into 8 versions across 22 repos before re-unification; survived a GitHub Actions incident that stranded 11 PRs for 14 h |
| `verify/verify-predicates.sh` | Assert a repo's verify gate actually exists and can fail | Found 6 lint scripts that had never run once |
| `verify/verify-floor-audit.sh` | Fleet-wide floor: every repo at or above the check baseline | 23/24 repos held at floor |
| `verify/find-false-needs.py` | Detect CI `needs:` that encode ordering, not dependency | Cut wall-clock on the worst pipeline by turning fake chains parallel |
| `templates/ci-npm.yml`, `ci-pnpm.yml` | The verify-calling CI workflow, npm/pnpm | |
| `templates/workflows/publish-reconciler.yml` | npm publishing without tokens: merge a version bump and the package ships (OIDC trusted publishing) | Powers [`ai-forms`](https://www.npmjs.com/package/ai-forms), [`threadkit`](https://www.npmjs.com/package/threadkit), [`bip-kit`](https://www.npmjs.com/package/bip-kit) |
| `templates/pre-commit` | The local half: tsc + eslint before any commit exists | |

Each script's header comment explains not just what it does but **which outage taught it** — read them like a post-mortem archive.

## Status

**v0 — honest extraction, not yet a product.** These are the canonical copies, adopted across the fleet via a reusable workflow. What v1 needs before you should run it blind: an installer, repo-shape detection, and docs written for someone who isn't me. Star/watch if you want that to exist — this repo is being built in public, decisions and dead ends included, on [orangecat.ch](https://orangecat.ch/profiles/maonakamoto).

## The story

The fleet this comes from is one person plus AI agents. The agents propose; the harness disposes. In one recent night, three npm packages were migrated to tokenless publishing, two production apps switched to a newly-published dependency, and one of the merges was co-authored by two different agents that had never been told about each other — the sweep merged it, the reconciler deployed it, and the health checks proved it live. The human's role was four security-key taps.

That's the thesis: **the constraint on a solo builder is no longer hands — it's judgment.** Encode the judgment in gates, and one person ships like a company.

## License

MIT
