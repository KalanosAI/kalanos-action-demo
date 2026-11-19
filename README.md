# Catch a broken robot recording before it merges

A small GitHub Actions demo for robotics and data engineers. A synthetic joint-position recording fails the PR check when timestamps repeat or move backwards and position samples are missing. Replace the corrupt recording with a known-good export and watch the same check pass.

| Run | Recording | Expected check |
| --- | --- | --- |
| Before | Repeated/out-of-order timestamps and NaN joint positions | **Red:** 1 episode unknown |
| After | Ordered timestamps and finite joint positions | **Green:** 1 episode pass |

**Why “unknown”?** The defective recording does not provide enough usable evidence for a pass. The policy explicitly fails on unknown, review, or blocked. The red check here is not a Review decision or an installer error.

## Reproduce in your repo

1. Create a repo from these files or fork this one. Enable GitHub Actions if your fork has it disabled. Keep the workflow on `main`.
2. Run the clean baseline: **Actions → Robotics dataset audit → Run workflow**, selecting `main`. Confirm one episode passes.
3. Follow the [recording guide](docs/DEMO-SETUP.md) to stage a red PR and then push the fix to the same PR.

## Read the result

The job summary shows the gate and decision counts. Download the **kalanos-audit** artifact for the generated `audit.json`. The report is retained after a gate failure if the grader produced it; a missing report is a reason to check the job log.

The Action runs the open-source Kalanos CLI on the runner. No MCP, Hub account, or API key is needed. This workflow intentionally publishes only the **synthetic** audit to GitHub Actions artifacts. Do not copy its artifact upload into a private-data workflow without reviewing what the report contains.

[Marketplace Action](https://github.com/marketplace/actions/kalanos-robotics-dataset-audit) · [Grader source](https://github.com/KalanosAI/kalanos) · [Kalanos Hub](https://hub.kalanos.ai)
