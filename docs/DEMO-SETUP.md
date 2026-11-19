# Kalanos GitHub Action demo setup

This demo uses a 200-row synthetic recording. It shows a data-driven CI failure and a data fix with the same grader, workflow, and policy. There is no need for a private dataset, a Hugging Face download, or branch protection.

## 1. Publish the demo repo

Create a new **empty** GitHub repository, `KalanosAI/kalanos-action-demo`, with no generated README. From the directory containing the supplied patch:

```bash
PATCH="$PWD/kalanos-action-demo.patch"
mkdir kalanos-action-demo
cd kalanos-action-demo
git init -b main
git apply --check "$PATCH"
git apply "$PATCH"
git add .
git commit -m "Add Kalanos robotics dataset audit demo"
git remote add origin https://github.com/KalanosAI/kalanos-action-demo.git
git push -u origin main
```

If you cloned the empty GitHub repo first, skip `mkdir`, `git init`, and `git remote add`. If you are working from a fork or a differently named repo, use its remote URL. Check that repository or organization policy allows the pinned Kalanos Action and standard GitHub actions.

Run **Actions → Robotics dataset audit → Run workflow** against `main`. Stop setup if this baseline is not green. The check assumes `main` is the default branch.

## 2. Stage the red check

Use a **branch in the demo repo** so you can push the repair to the same PR later.

```bash
git switch -c demo/repair-recording
cp examples/defective-recording.csv fixtures/demo/recording.csv
printf '%s\n' '# Demo recording submission' '' \
  'Synthetic joint-position capture used to demonstrate the PR audit.' \
  > fixtures/submission.md
git add fixtures/demo/recording.csv fixtures/submission.md
git commit -m "Submit synthetic robot recording"
git push -u origin demo/repair-recording
```

Open a PR into `main`, titled **Repair a synthetic robot recording**. Wait for **Kalanos dataset audit** to go red. It should show **1 unknown**, not an installation or path error. If the run says **error** or has no report, resolve that before filming.

**Record the before take now.** Save the PR URL and failed run URL. Capture the red check, the summary counts, and the data defect (two or three CSV rows showing repeated timestamps or a `nan` value). You can open the synthetic `audit.json` afterward for supporting evidence. The report may include more than one quality signal; use the actual wording it emits rather than assuming a particular finding ID.

## 3. Push the fix to the same PR

```bash
git restore --source=origin/main -- fixtures/demo/recording.csv
git add fixtures/demo/recording.csv
git commit -m "Restore valid timestamps and joint samples"
git push
```

The submission note keeps the PR open with a visible diff while the known-good recording is restored. The note is outside `fixtures/demo/` and is never graded. Wait for the new run and confirm **1 pass, 0 unknown**. Do not rerun the old failed commit or weaken the policy.

**Record the after take.** Show the repair commit and the green check on the same PR. Save the passing run URL. Leave the PR open until recording is finished. The previous run is still available via its direct Actions URL.

## What to send for recording

- Public repo and PR URLs.
- Failed and passing Actions run URLs, labeled **before** and **after**.
- Optional: the logged timestamps or the relevant JSON snippet to show in the evidence close-up.

Suggested 25–35 second edit: **red check → evidence → corrected data → green check**. Lead with the red check, not installation. If you show the JSON artifact, open it ahead of the take to avoid recording the browser downloads menu.

## Scope and verification

- Only `fixtures/demo/` is graded. The deliberately defective source copy under `examples/` is not graded until you stage it.
- The Action is pinned to its v1.0.1 release commit; the grader is pinned to 0.9.3. Both runs use the same default scope and tier and `fail-on: blocked,review,unknown`.
- These fixtures are derived from the [published hosted Action smoke workflow](https://github.com/KalanosAI/kalanos-action/blob/v1.0.1/.github/workflows/smoke.yml). Its [validation record](https://github.com/KalanosAI/kalanos-action/blob/v1.0.1/VALIDATION.md) confirms pass for the clean recording and a gate failure with 1 unknown for the defective recording. This starter uses the same CSV values; the full demo PR still needs its own hosted runs.
- A passing audit does not prove the recording will train a successful policy.
