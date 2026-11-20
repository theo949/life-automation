---
name: "todo-offline"
description: "Helper skill loaded by todo-review and todo-card when the user types #offline on a card, and at the wrap-up when offline jobs are queued. Not for direct use."
---

# Offline jobs (helper)

This helper is loaded with the Skill tool by `todo-review` or `todo-card`, once per run, when the user types `#offline` on a card (to queue the job) and at the wrap-up when the queue is not empty (to start it). It is not started by the user. Everything in the core skill still applies (typed menus with circled numbers, no recaps, 12-hour times, no narration).

An offline job is research or writing the user does not want to wait for. The review queues it and carries on. After the wrap-up, ONE scheduled task does all the queued jobs, one at a time, and puts each result on its card.

## Queueing a job (during the walk)
- **Trigger:** the user's reply contains `#offline` (any case, anywhere in the text) on a card. Everything else in the reply is the job, in the user's own words, with the tag removed. Example: `#offline compare standing desks under $300`. The user never has to say how the result should look: the task decides (see **Result**).
- **Plan check:** offline jobs need scheduled tasks. If the settings say the plan is manual, do not queue. Reply with `Offline jobs need scheduled tasks, and this setup doesn't have any.` and the typed question `Do it now instead?` with the menu `① Do it now · ② Skip it`, then follow the answer.
- **Empty job:** if the reply is only `#offline`, ask one line `What should I do offline?` and wait.
- **Queue entry (kept in your notes, no tool call):** the card ID, card name, card URL, the job text, and the few lines of the card the job may need (links under Documents, people and emails, event dates). A clear snooze or mark done typed in the same reply is handled by the core as usual; only the job goes into the queue.
- **Reply:** a one-line note at the top of the same reply that holds the next card or the check-in: `Queued for after the wrap-up: <the job in 8 words or fewer>.` Nothing else. The job is never started during the walk, so nothing runs next to the review.

## Starting the jobs (wrap-up)
The wrap-up message lists the queue in one line: `🌙 Offline: <each job, one short clause>, once we're done here` (the wrap-up icon line, see Step 5 in `todo-review`). Then, after the Write queue flush report is in and before the closing message, create ONE scheduled task with the same scheduling tool the router uses for the helper tasks (load it with ToolSearch if it is deferred):
- name `Offline jobs <YYYY-MM-DD>`, a one-time run about 3 minutes from now (`run_once_at`, UTC), no folders, no computer needed;
- the prompt below, filled in with the jobs (all of them in the order the user queued them);
- leave the approval setting unset. Read it in the result: if the task does not approve automatically, add one line to the closing message: `Heads up: the offline task may stop and wait for your approval. Set "Offline jobs <date>" to Automatically approve in its settings if your organization allows it.`
- if creating the task fails, the closing message says `Offline jobs didn't start: <reason>. Say "retry" and I'll give it another go.`

A stopped review still starts the jobs the user already queued. A review closed before the wrap-up does not start them.

## The task prompt
Write it as a complete instruction, because the task starts with no memory of this chat:

"Offline jobs for the user of the Life Automation Trello board. Do the jobs below one at a time, in order, never two at once. Do not ask questions: if something is unclear, pick a sensible default and write the assumptions at the top of the result. Research and writing only: never send an email or message, never move, snooze, rename or archive a card, and change nothing except what is listed below.

For each job:
1. Read the card (trelloReadCard get) and use the context lines.
2. Do the job (web search, linked files, thinking). Prefer reputable sources and link them. For health, legal or money questions say the result is general information, not advice, and list questions to ask a professional.
3. Choose where the result goes. A short answer (a fact, a recommendation, a yes or no with the reason, about 10 lines or fewer) goes in as a Trello comment. Anything longer (a comparison, a table, a list of options, a summary of several sources) goes in a Google Doc. When unsure, make the doc. The user's own words win if they asked for a format.
4. A Google Doc: create it in the user's Google Drive with the Google Drive connector, titled with a short job title; short answer first, then the details, then the sources with links, in plain language. If Google Drive is not connected, make a Claude Doc instead.
5. Deliver: add a Trello comment `**YYYY-MM-DD:** Offline result: <one to three lines>` and, for a doc, the link. For a doc, also read the card once and add one line `- <doc title> (Google Doc): <link>` under `## Documents` in the `--- 🤖 Claude metadata ---` block, set `Last updated` to today and leave every other character unchanged. A comment-only result changes no metadata.
6. If a job cannot be finished, add the comment `**YYYY-MM-DD:** Offline job couldn't finish: <short reason>` and go on to the next job.

Finish with one line per job: `done: <card name>, <doc link or comment>` or `failed: <card name>, <reason>`. Do not write to memory files.

Jobs:
<for each job: card ID, card name, card URL, the user's words, the context lines>"

## Result (what the user sees later)
Nothing new is needed in the brief: the result comment is news on the card, so tomorrow's scan shows it like any other new comment, and a doc is a linked file the scan already watches.