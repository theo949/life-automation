---
name: "todo-prep"
description: "Life Automation helper (paid plans): does the slow read-only prep for the daily review ahead of time and writes it to the Review prep tab. Runs on the daily schedule 45 minutes before the review."
---

# Review prep (helper, paid plans only)

Does the slow, read-only prep for the daily review ahead of time and writes it to the **Review prep** tab of the Google Sheet "Life Automation Log", so the review (`todo-review`) starts fast. It runs silently as a scheduled task 45 minutes before the review, after `todo-prioritizer` (75 minutes before) and `todo-discovery` (60 minutes before). Non-paid plans never run it: the review preps live.

Never send, draft, label, archive or delete any email. Never create or change calendar events. Never write to Trello (no comments, metadata, moves, labels, snoozes, checklist changes). Never message anyone. Never create scheduled tasks; the only scheduled-task change allowed is aligning this task's own cron in STEP 0. The only thing you may write is the **Review prep** tab (cleared and rewritten each run, STEP 5); read the other tabs only. Keep IDs and dates as text (prefix with an apostrophe). **Privacy:** the sheet gets pointers, flags and codes only: no subjects, names, amounts or free text.

## STEP 0: Settings and schedule check
Read `/areas/life-automation.md`. If missing, or paused, say so in one line and stop. Use the saved time zone and the saved board and sheet names. Find the board by name in Trello and the sheet by title in Drive; never use stored IDs.
Mondays only: list_triggers (recurring, enabled; follow next_cursor) and find "Life Automation: review". This task (named "Life Automation: prep") must run exactly 45 minutes before it, same days. If its cron differs, update_trigger it (saved time zone, hour and minute computed minus 45) and continue.

## STEP 1: Context (read-only)
a) Read `/areas/life-automation-learnings.md`. Lines starting `Gmail discovery:` are exclusions; respect any line saying a topic must not be brought up unless asked.
b) Read these tabs: **Card index** (`card_id, title, list, people_emails, thread_ids_hex, last_updated, refreshed_date`), **Card news** (`found_date, thread_id, last_msg_date, card_id, sender_domain, category, needs_action, notes`), **Proposals** (`found_date, thread_id, card_id, sender_domain, category, suggested_list, status, outcome_date, outcome_list, notes`). If the newest `refreshed_date` in Card index is older than yesterday, the prioritizer did not run: do your own Gmail search for every card's people instead of trusting Card news, and set the final status to `partial`.
c) Work out today's date in the saved time zone (python3 zoneinfo) and the current UTC timestamp for `prepared_at`.

## STEP 2: Read the board
Read all lists except `Done` and `Proposed`. **Review order** (the order the review walks the cards): the working lists are `TO DO` and `In Progress`; start at `In Progress` and then `TO DO`; within a list, top to bottom by card position. Due dates do not change this order. Cards in the three Snoozed lists are not prepped as cards; only check their metadata for stale FUTURE meetings (STEP 3 calendar check) and write a row for a snoozed card only if it has such a meeting. **Cap:** full prep for the first 35 working cards in review order; for the rest write a row with `reason_codes=no_prep_cap` and `suggested_action=none`, and count them as cap overflow.

## STEP 3: Per-card prep (read-only)
Use subagents in batches of about 5 cards if the Agent tool is available, otherwise batch independent tool calls. For each card read the full card (description, comments), its checklists directly (trelloReadChecklist; the plain card read can show none), and its `--- 🤖 Claude metadata ---` block. Record:
- **deadline:** the earliest of the Trello due date and any hard external deadline you find (bill or statement due date, filing deadline, appointment), as YYYY-MM-DD, empty if none.
- **new_thread_ids_hex and news:** Gmail thread IDs (hex) involving the people in the card's metadata, newer than the block's `Last updated`. Take them from the Card news rows for this card_id with `last_msg_date` later than `Last updated`; do your own read-only Gmail search for the card's people only when the card is missing from Card index, Card index is stale (STEP 1b), or the card has `Wake on reply from`. Read each new thread with get_thread to judge whether it needs action, but write only IDs and codes. news=yes if any new thread exists.
- **calendar_change:** none, new (a new event involving the card's people since `Last updated`), moved or deleted (a future meeting listed in `Events:` no longer exists at that time; search Google Calendar for a moved version before saying deleted). Never flag past meetings for removal. If Google Calendar is not connected, write none.
- **reason_codes** (comma-separated, only those that apply): new_reply, new_email, calendar_changed, meeting_past_no_outcome (a connected meeting in the last 14 days whose line in `Events:` has no (done), (not done) or (didn't happen)), missing_travel (only when a home address is saved: a future in-person meeting within 14 days with no event titled like "Travel" ending at its start or up to 15 minutes before; judge in-person from title, location and thread), checklist_outdated, title_outdated, description_outdated, linked_file_news (for linked Google Sheets, Docs or Slides: read comments with get_spreadsheet commentsIncluded=true and Drive read_file_content includeComments=true; new or changed threads since `Last updated` count; cap 10 linked files per run), deadline_soon (deadline within 7 days), waiting_7d, no_metadata, no_change.
- **suggested_update:** none, or a comma list of checklist, title, description, when developments made them outdated. Do not write what should change.
- **suggested_action** (one code): send_email, snooze_until_reply (waiting on someone after we asked or emailed and nothing came back), snooze_until_meeting (the ONLY open item is a future, not stale meeting that is not today and not later than 1 day before the card's deadline, with no unchecked checklist items, no reply awaited, no email to send, no missing travel event), schedule_meeting, update_card (checklist, title or description update only), mark_done (only when clearly finished), nudge (waiting 7+ days with no reply), none, needs_review (anything unclear or unusual). Never decide a snooze date.
- **confidence:** high only when the evidence is direct (e.g. a reply that plainly answers the question); med if reasonable inference; low if guessing.
Cards without a metadata block: write `no_metadata`, still fill deadline and any news found; leave enrichment to the live review.

## STEP 4: Proposals
Read the `Proposed` list (cards with their `--- 🤖 Claude proposal ---` blocks). One row per proposal (`item_type=proposal`, `item_id` = the card's object id). Fields: list = the block's Suggested list; deadline = Suggested due (empty if none); new_thread_ids_hex = the block's Thread ID; `proposal_suggestion`:
- **add** (default: a real task for the Suggested list);
- **merge** (only when the block's Possible match is a Trello card and the email clearly belongs on that card by people AND topic; put that card's object id in `merge_target_card_id`);
- **dismiss** (only when a `Gmail discovery:` learning applies, or the Proposals tab shows that the sender_domain or category group has at least 3 resolved rows (accepted, dismissed or merged) with at least 70% dismissed and the email has no amount, due date or direct personal request, or the user's own message is already the last one in the thread);
- **completed** (the thread shows the user already handled it, e.g. their reply is last and answers it, or the amount was paid).
`reason_codes` from: possible_match, learned_dismiss, learned_accept, already_handled, due_soon, personal_request, old_7d (Found 7+ days ago), no_signal. `confidence` as above. Suggestions only: never move, label, archive or change proposal cards.

## STEP 5: Write the cache
Compute all rows first. Then, in ONE update_values call (so the tab is never half written) after one batch_clear_values on `'Review prep'!A2:P500`, write from A2 these rows in this column order: `item_type, item_id, list, deadline, new_thread_ids_hex, calendar_change, news, suggested_update, suggested_action, proposal_suggestion, merge_target_card_id, reason_codes, confidence, prepared_at, prepared_date, notes`.
- **First row = the meta row:** `item_type=meta`, `item_id=run`, `news` = status (`ok`; `partial` if Card index was stale, any read failed or the cap overflowed; `failed` if the board read returned nothing), `prepared_at` = current UTC time as an ISO string (e.g. 2026-10-19T05:14:00Z), `prepared_date` = today's date in the saved time zone (YYYY-MM-DD), `notes` = counts like `cards=31;proposals=4;snoozed_meeting_checks=2;card_index_refreshed=YYYY-MM-DD;cap_overflow=0;errors=none` (no names or titles).
- Then all card rows in review order (`item_type=card`), then the proposal rows (`item_type=proposal`). Fill `prepared_at` and `prepared_date` on every row.
- If the board read failed, still replace the tab with only the meta row (status `failed`) so the review knows there is no prep today.

## STEP 6: Final reply (the run record, 2-5 lines)
Status, cards prepped, proposals prepped (add, merge, dismiss and completed counts), cap overflow, Card index age, whether the Monday schedule check ran and changed anything, any error. Do not email or message anyone.
