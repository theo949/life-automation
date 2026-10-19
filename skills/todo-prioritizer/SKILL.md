---
name: "todo-prioritizer"
description: "Life Automation helper: ranks the cards on the user's Trello board and refreshes the Card index tab of the log sheet. Runs on the daily schedule 75 minutes before the review, or inline from /todo on non-paid plans."
---

# Card prioritizer (helper)

Ranks the open cards within each working list and refreshes the **Card index** tab that the discovery and prep helpers and the review read. It runs silently: on paid plans as a scheduled task 75 minutes before the review, on non-paid plans inline at the start of `/todo`.

Never send, draft or delete any email. Never message anyone. Never move a card to another list, archive or delete a card, or edit titles and descriptions. The only Trello change is card **positions inside a list**. The only sheet you may write is the **Card index** tab.

## Step 0: Settings
Read `/areas/life-automation.md`. If missing or paused ("paused" = true on a scheduled run), say so in one line and stop. Use the saved time zone for "today" and the saved board and sheet names. Find the board by name in Trello and the sheet "Life Automation Log" by title in Drive. Never use stored IDs. Keep IDs and dates as text when writing to the sheet (prefix with an apostrophe).

## Step 0b: Schedule check (Mondays only, scheduled runs only)
Skip on other days and on inline runs. On Monday: list_triggers (recurring, enabled; follow next_cursor) and find the task named "Life Automation: review". This task (named "Life Automation: prioritizer") must run exactly 75 minutes before it, same days. If its cron differs, update_trigger it (hour and minute computed minus 75, in the saved time zone) and carry on.

## Step 1: Read the board
Read the lists. Working lists are `TO DO` and `In Progress`. Ignore `Done`, `Proposed` and the three Snoozed lists everywhere in this skill (never reorder or comment on them). For each open card in the working lists get: id, list, position, due date, description, last activity.

## Step 2: Reorder within each working list
Never move a card to another list. Rules:
- **Primary: due date.** Overdue first (most overdue first), then soonest due, then cards with no due date.
- **Secondary, for cards without a due date:** recently active before dormant (smaller days since last activity first), then newer before older.
- Stable tie-break: existing order.
- Apply with trelloWriteCard (position only), only where the rank changes. Skip a list with fewer than 2 cards.

## Step 3: Refresh the Card index tab
The tab is a current-state snapshot, overwritten each run. Columns: `card_id, title, list, people_emails, thread_ids_hex, last_updated, refreshed_date`.
- If the board read failed or returned no cards, leave the tab as it is and say so in the final reply.
- Otherwise compute all rows first, then batch_clear_values on `'Card index'!A2:G2000` and write one row per open card in `TO DO`, `In Progress` and the three Snoozed lists, from A2, in one update_values call.
- `card_id` = the card's object id. `title` = card title. `list` = list name.
- `people_emails` = the lowercase email addresses in the People lines of the card's `--- 🤖 Claude metadata ---` block, comma-separated (empty if no block).
- `thread_ids_hex` = the Gmail thread IDs in the block's Emails lines, as lowercase 16-character hex IDs, comma-separated, deduplicated. Gmail links come in several shapes: (a) URLs ending in `thread-f:<decimal id>`: convert the decimal to hex with `python3 -c "print(format(int('<decimal id>'),'x'))"`; (b) URLs whose fragment ends in a 16-character hex token (`#all/<hex>`, `#inbox/<hex>`, `#sent/<hex>`): use that token as is; (c) `thread-a:r<number>` URLs: ignore. Do the conversions in one python run, not by hand.
- `last_updated` = the block's `Last updated` date (empty if no block). `refreshed_date` = today in the saved time zone, YYYY-MM-DD.

## Step 4: Final reply (the run record, 2-4 lines)
Cards reordered per list, rows now in the Card index, whether the Monday schedule check ran and changed anything, any error. If nothing changed: "No reordering needed today." Do not email or message anyone.
