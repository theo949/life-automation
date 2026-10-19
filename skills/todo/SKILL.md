---
name: "todo"
description: "Entry point for Life Automation. Use when the user types /todo or asks for their daily task review, settings, pause or uninstall."
---

# /todo: Life Automation router

This skill only decides where to go. It does no review work itself.

## Step 1: Find the saved config

1. Read the memory file "Life Automation settings" (path: /areas/life-automation.md). It holds: review time, days, time zone, email lookback, home address (optional), plan type (scheduled or manual), Trello board name, log sheet name.
2. If the file has a `setup_stage` that is not `done` (setup was interrupted, for example by a message limit), invoke `todo-setup` to resume it. Otherwise, if the file has a board name and a sheet name, treat the user as configured.
3. If the file is missing or incomplete, run the lookup fallback before concluding they are new:
   - Trello: search for a board named "Life Automation" (trelloSearch).
   - Drive: search for a sheet titled "Life Automation Log".
   - If BOTH are found, rebuild the memory file with what you can recover (names, plus anything visible in existing scheduled tasks via list_triggers), tell the user in one line that you restored their settings, and treat them as configured.
   - If only one is found, or neither, invoke the `todo-setup` skill. Pass along what was found so setup reuses it instead of creating duplicates.

## Step 2: Route

Look at what the user typed after /todo.

| Input | Action |
|---|---|
| nothing | Not configured: invoke `todo-setup`. Configured on a paid plan: invoke `todo-review`. Configured on a non-paid plan (settings say manual): first run the discovery and prioritizer steps inline (see Manual runs below), then invoke `todo-review`. |
| `settings` | Show the current settings in plain language. Ask what to change. Apply changes to the memory file, and if time, days or time zone changed on a paid plan, update the three helper tasks and the review task (see Timing below). |
| `pause` | Paid plans: disable the four scheduled tasks. Record "paused" in memory. Say how to resume. |
| `resume` | Re-enable the four scheduled tasks. Record "active". |
| `uninstall` | Explain what will happen first, then wait for a clear yes: scheduled tasks are deleted; the Trello board, the log sheet and the memory file are NOT deleted unless the user asks for each one by name. Delete only what they confirm. |
| `setup` | Invoke `todo-setup` even if configured, in repair mode (it checks what exists and only creates what is missing). |

## Timing (paid plans)

The review runs at T. The helpers run before it, in this order, because discovery and prep read the Card index tab that the prioritizer writes:
- prioritizer at T minus 75 minutes
- discovery at T minus 60 minutes
- prep at T minus 45 minutes

## Manual runs (non-paid plans)
There is no schedule, so before the review do the helpers' work inline, in this order: (1) invoke `todo-prioritizer` to rank the cards and refresh the Card index tab, (2) invoke `todo-discovery` to scan Gmail over the saved lookback and fill the Proposed list. Skip prep. Tell the user in one line that this takes a minute or two, then start the review.

## Rules

- Never start a review for a user who is not configured.
- Never delete the user's board, sheet or memory without a named, explicit yes.
- Speak in plain words. Do not mention skills, memory files or tool names unless the user asks.