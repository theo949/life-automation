---
name: "todo-logged"
description: "Shortcut that runs the Life Automation daily review with timing logs. Use only when the user types the exact command /todo-logged, never for /todo with an argument."
---

# /todo-logged: the review, with a timing log

This skill is only a shortcut for `/todo debug`. It does no review work itself and never copies the review rules, so it cannot drift from the real review. Use it only when the user typed the exact command `/todo-logged`. When the user typed `/todo` followed by an argument (for example `/todo debug`), this skill is not involved: the `todo` router handles it.

## Steps
1. Your first text is exactly this line, as plain chat text: `Rolling up my sleeves for your daily review…`. Write nothing else before it and run no other call first. Then your first call is to invoke the `todo` skill with the argument `debug`, exactly as if the user had typed `/todo debug`. The `todo` router does not write the line again; it takes the start stamp itself in its first batch, turns on logging and sends a configured user to `todo-review` as usual. An unconfigured user goes to setup as usual, and logging does not apply to setup.
2. When `todo-review` runs with Debug mode ON, it follows its **Debug mode** section: it logs every step, saves the log to the Debug tab of the log sheet, and ends with a short timing summary.

## Rules
- Typing `/todo debug` is the exact real experience: this shortcut adds one skill-load turn before the router starts, so the first stretch of its timings is a few seconds longer. Say this once in the timing summary when the run started through this shortcut.
- Logging changes nothing about how the review runs: the same batching, the same prefetch, the same questions. It only adds logging.
- The review may ask whether to do a quick or a full review. Do not answer it for the user and do not skip it: ask it as usual. `todo-review` logs the chosen mode (quick or full) in the Debug tab, so both modes can be timed.
- Logging lasts for this run only. The next plain `/todo` run is a normal run with no logging.
- Never log card titles, email text, names or addresses. Only card IDs, short step codes and counts.