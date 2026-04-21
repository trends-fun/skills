# IAO Cron Discovery Guide

Use this document when users ask for automatic/scheduled discovery of new IAO projects.

## 1) Ask-to-create policy

If cron-job capability is available:

- Ask once when this policy is first loaded in a session.
- Ask again when users explicitly mention:
  - discovering new IAO projects
  - automatic polling/monitoring
  - scheduled checks/cron reminders

Prompt:

- `Do you want me to create a cron job to fetch and analyze new IAO projects automatically?`

## 2) State model

Maintain one state variable only:

- `last_run_unix` (Unix integer seconds, never ISO format)

Suggested local state file:

- `your-workspace/iao-project-cron-state.json`

Example:

```json
{"last_run_unix": 1712572800}
```

## 3) Runbook (strict order)

1. Sync state
- Read memory/state file.
- If `last_run_unix` is missing or invalid:
  - compute `now_unix`
  - write `last_run_unix = now_unix`
  - exit silently (no user interruption)

2. Get current timestamp
- Compute `now_unix` as Unix integer seconds.

3. Fetch projects in window
- Use:

```bash
trends-skill-tool iao project list --count 25 --start-at <last_run_unix> --end-at <now_unix>
```

4. Analyze result
- If list is empty (`[]` or length `0`):
  - silent output and end.
- If list has items:
  - for each project show:
    - `url`
    - `title`
    - `referenceUrl`
    - `submitter`
    - `description`
  - evaluate narrative value:
    - meme potential
    - heat/attention potential
    - community fit
  - if alpha value exists:
    - proactively ask whether user wants to create an IAO model token.

5. Update state
- Persist `last_run_unix = now_unix` to memory/state file.

## 4) Output contract

- No new projects: silent output, no disturbance.
- New projects found: output full analysis and actionable recommendation.

