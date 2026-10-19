---
name: "todo-discovery"
description: "Life Automation helper: scans the user's Gmail for emails that need action and adds each as a card in the Proposed list. Runs on the daily schedule 60 minutes before the review, on setup, or inline from /todo on non-paid plans."
---

# Task discovery (helper)

Finds emails that need action from the user and adds each one as a card in the Trello list `Proposed`, so the review can triage them. It runs silently: on paid plans as a scheduled task 60 minutes before the review, during setup (first scan), and on non-paid plans inline at the start of `/todo`, after `todo-prioritizer`.

This run never asks questions. Never send, draft, label, archive or delete any email. Never touch cards outside `Proposed` (reading is fine). Never message anyone. The only change to a card is on cards you create yourself in `Proposed`: applying existing board labels (STEP 3a); never due dates or members. In the sheet "Life Automation Log" you may write the **Proposals** tab (append and update only), append to the **Card news** tab, and read the **Card index** tab (written by the prioritizer; never write it). Do not touch other tabs (including **Review prep**). Keep IDs and dates as text (prefix with an apostrophe).

## STEP 0: Settings and schedule check
Read `/areas/life-automation.md`. If missing, or paused on a scheduled run, say so in one line and stop. Use the saved time zone for "today", the saved lookback, the board name and the sheet name. Find the board by name in Trello and the sheet by title in Drive. Never use stored IDs.
Mondays only, scheduled runs only: list_triggers (recurring, enabled; follow next_cursor) and find "Life Automation: review". This task (named "Life Automation: discovery") must run exactly 60 minutes before it, same days. If its cron differs, update_trigger it (saved time zone, hour and minute computed minus 60) and continue.

## STEP 1: Context
a) Read `/areas/life-automation-learnings.md`. Lines starting `Gmail discovery:` are exclusions (senders, subjects, topics never to propose). Also respect any line saying a topic must not be brought up unless asked.
b) Find the list `Proposed` on the board. If missing, create it at the right end. Read the board's existing labels (name and id) for STEP 3a.
c) **Known threads.** Do NOT read all cards. Build the known set from every `thread_id` in the **Proposals** tab and the `thread_ids_hex` column of the **Card index** tab. Gmail's API gives hex thread IDs, but links stored on cards use the decimal form (`thread-f:<decimal>`; convert with `python3 -c "print(int('<hex>',16))"`). After STEP 2 produced the qualifying candidates, for each one not already known, run trelloSearch with the DECIMAL ID twice (plain, and with ` is:archived` appended) and once with the hex ID. Any hit means known: skip it. Also load the full **Card index** tab for STEP 2b. If it is empty or its newest `refreshed_date` is more than 3 days old, say so in the final reply and, for this run only, read all open cards on the board (the read is very large: it is saved to a file, then use jq or grep), grep thread IDs and note open card titles and People.
d) **Learn from past proposals** (Proposals tab, columns `found_date, thread_id, card_id, sender_domain, category, suggested_list, status, outcome_date, outcome_list, notes`).
   1. Sync outcomes: for every row with status=open, read its card. If the card is open and in a list other than `Proposed`: status=accepted, outcome_list=that list, outcome_date=today. If it is archived while in `Proposed` and its thread ID appears on another card: status=merged. If it is archived and no other card has the thread ID: look at its comments. A comment starting "Already completed" means status=completed; otherwise status=dismissed. If still open in `Proposed`: leave open (after 14 days set status=ignored). Update the row in place and note the sync in the final reply.
   2. Learned weighting (only rows with status accepted, dismissed or merged; ignore open, ignored and completed): group by `sender_domain` and by `category`. If a group has at least 3 resolved rows and at least 70% are dismissed, treat it as low value: do NOT propose it, UNLESS the email has an amount or due date (bill, fine, fee, deadline) or is a direct personal request from a named person; count these under "skipped: learned". If a group has at least 3 resolved rows and at least 70% accepted, always propose it when it otherwise qualifies. Fewer than 3 rows: no change. Never use a single card's status to suppress anything. List each active learned rule (group, n, % dismissed or accepted) in the final reply. "completed" rows count as neither.
   3. Privacy: in the sheet write no subjects, names or amounts. `sender_domain` is the domain only; `category` is a generic word such as bill, appointment, personal-request, job, medical-portal-message, admin, reply-owed. For health or other sensitive items write sender_domain `[redacted]`.

## STEP 2: Search Gmail (read-only)
Window: `newer_than:3d` (covers weekends; the known set handles overlap). **First scan** (the Proposals tab is empty, or setup asked for it): use the saved lookback instead (1 week = `newer_than:7d`, 2 weeks = `newer_than:14d`, 1 month = `newer_than:30d`). Window guard: if the newest `found_date` in the Proposals or Card news tab is more than 3 days old (the task did not run), widen the window to cover the gap. Search inbox mail excluding promotions, social and forums, plus unread mail, plus sent mail (to see where the user already replied). Page through all results (the first page is only 50 threads). Read each candidate thread in full before judging.
Propose a task only if at least one is true: a request or question addressed to the user that needs their answer or action; a bill, invoice, fine or fee with an amount or due date; a deadline, form or document to submit; an appointment, meeting or delivery that needs preparation or a decision; a reply the user owes (the last message is from someone else, contains a question, and has waited 2+ days).
Do NOT propose: newsletters, promotions, marketing, receipts or confirmations needing no action, automated notifications (shipping, security alerts unless action is required), social notifications, FYI or CC-only threads, threads where the user's own message is the last message, threads in the known set, anything excluded by the learnings or the learned weighting.
**Cap:** at most 10 new proposals per run, and at most 15 on the first scan. If more qualify, keep the most urgent (soonest deadline, then newest) and mention the overflow count in the final reply.

## STEP 2b: Match against existing cards
Before creating any proposal, compare each qualifying thread with the Card index. A CLEAR MATCH needs both: (a) the sender's email (or a recipient's, on a sent thread) is in the `people_emails` of exactly one open card, or in several cards but only one has a title covering the same topic; and (b) the thread's topic clearly fits that card's title. For a clear match: do NOT create a proposal; append a row to **Card news** (STEP 3b) with needs_action=yes. If the person matches but the topic is unclear, or the person is on several cards and none clearly fits, treat it as unmatched and create the proposal, naming the candidate cards (Trello links) in "Possible match". Threads with no person match are unmatched. Never comment on, edit or move existing cards. Cards missing from the Card index (health cards are deliberately excluded) can never be matched.

## STEP 3: Create proposals
For qualifying threads that were not a clear match. Before creating each, re-check `Proposed` for the same Thread ID. Create the card in `Proposed` with a short, neutral, imperative title (e.g. "Pay phone invoice (due Oct 20)"; no sensitive details in titles) and this exact block as the description:

```
--- 🤖 Claude proposal ---
Found: YYYY-MM-DD
From: Name <email>
Gmail thread: <URL>
Thread ID: <id>
Why: <one line: what is needed from the user, and by when>
Suggested list: <TO DO or In Progress>
Suggested due: <YYYY-MM-DD or none>
Labels applied: <exact label names applied in STEP 3a, comma-separated, or none>
Possible match: <Trello card link if an existing card clearly covers the same subject or people, else none>
--- end ---
```

Suggested list: `In Progress` if it is a conversation the user is already actively handling (they replied recently); otherwise `TO DO`. Do not set the card's due date or members.
After each card is created (and labelled, STEP 3a), append one row to the Proposals tab: found_date, thread_id (hex), card_id (the card's object id), sender_domain, category, suggested_list, status=open, empty outcome_date and outcome_list, notes ("labels: N" with the number of labels applied, no label names).

## STEP 3a: Labels on new proposals
Save the user the sorting, but err on the side of safe:
- Use only labels that already exist on the board (STEP 1b). Never create, rename or delete a label.
- Apply a label only when you are PRETTY SURE it fits: the label's meaning is clear from its name and the thread's content (not just the sender's name or domain) plainly belongs to that category. If unsure, apply none. A card with no label is a fine result; a wrong label is worse than none. The same bar applies to every label.
- At most 2 labels per card. Apply them with trelloWriteCard on the card you just created (set only the labels). If applying fails, carry on, write "none" in the block's "Labels applied" line and mention the error in the final reply.
- Update the block's "Labels applied" line to match what was actually applied.

## STEP 3b: Card news
For each clear match from STEP 2b, and also for non-qualifying threads (FYI, confirmations, receipts) that involve a person on an open card, append one row to the **Card news** tab (columns `found_date, thread_id, last_msg_date, card_id, sender_domain, category, needs_action, notes`): found_date (today), thread_id (hex), last_msg_date (date of the thread's last message, YYYY-MM-DD), card_id (the matched card), sender_domain, category (generic word), needs_action (yes or no), notes (short, neutral: no subjects, names or amounts). Skip a row if one with the same thread_id and last_msg_date already exists. Cap 40 rows per run. Do not create Trello cards for these.

## STEP 4: Final reply (the run record, 3-8 lines)
Threads scanned, proposals created (neutral titles), labels applied (N cards, each neutral title with the label names; or "no labels applied"), skipped counts by reason (including "skipped: learned"), cap overflow, outcomes synced (N accepted, N dismissed, N merged, N completed) and any active learned rules, "matched to existing cards: N threads added to Card news (K need action)" with the neutral titles of the matched cards, Card index age in days (or the fallback note), whether the schedule check ran (Mondays) and changed anything, any error. If nothing qualified: "No new tasks found." plus the outcome sync line if anything changed. On the first scan during setup, report the proposal count in one line so setup can show it.
