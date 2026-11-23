---
name: parallel-agent-queue
description: Use when coordinating several AI coding/research agents (Codex, Claude Code, Grok, Gemini CLIs or API runs) on one machine — keeping them busy from a job queue with safe limits, status files and usage planning.
---

# Parallel Agent Queue

Keep multiple AI agents working without idle gaps or runaway resource use. By The Ask Why Company.

## Layout
```
lanes/
  run-<engine>.sh NAME   # one runner per engine: writes NAME.status (START …), runs the agent with NAME.md as the prompt, saves NAME.log and NAME.last.md, appends EXIT <rc>
  queue/<engine>/*.md    # job files waiting; oldest file runs first (touch -t an old date to jump the queue)
  queue/done/            # moved here when started
  queue-runner.sh        # loop: count running jobs per engine (status files with START but no EXIT), start the oldest queued job when below the limit
```

## Rules
- **Limits per engine** (e.g. 4–6 for a large plan, 1–2 for a small one). Count by status files, not process names.
- **Guards:** pause when free disk drops below a threshold; each job uses its own worktree/scratch folder under the project, not /tmp.
- **Stale jobs:** a START with no running process → append `EXIT stale-no-process` so the slot frees.
- **Runners are detached** (nohup) so restarting the queue runner never kills running jobs. Editing a script can drop its execute bit — `chmod +x` after edits.
- **Usage planning:** know each engine's rolling limits and reset days; heavy models drain short windows fast, so default to a mid model and save the strongest for hard tasks; use expiring credits first.
- **Every job file** states: goal, repo/branch, what not to do (no deploys, no merges, no secrets), how to verify, and where to write its summary.
- **Never leave the board idle:** when a slot frees, the next queued job starts; refill the queue before going away.
- Secrets reach agents via environment variables loaded from the OS keychain at launch — never in job files.
