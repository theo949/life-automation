---
name: "todo-prioritizer"
description: "Life Automation helper: wakes snoozed cards whose date has arrived, rolls the Snoozed lists forward, ranks the cards on the user's Trello board and refreshes the Card index tab of the log sheet. Runs on the daily schedule 75 minutes before the review, or inline from /todo on non-paid plans."
---

# Card prioritizer (helper)

Wakes the snoozed cards whose date has arrived, rolls the Snoozed lists forward, ranks the open cards within each working list and refreshes the **Card index** tab that the discovery and prep helpers and the review read. It runs silently: on paid plans as a scheduled task 75 minutes before the review, on non-paid plans inline at the start of `/todo`.

Never send, draft or delete any email. Never message anyone. Never archive or delete a card, or edit titles. The Trello changes you may make are card **positions inside a list** and the two date-based moves of Step 1b (waking cards whose snooze date has arrived, and rolling the Snoozed lists forward), including the snooze lines they remove from the card's metadata block, the `Woke on` line a wake adds, and their dated comments. The only sheet you may write is the **Card index** tab.

## Step 0: Settings
Read `/areas/life-automation.md`. If missing or paused ("paused" = true on a scheduled run), say so in one line and stop. Use the saved time zone for "today" and the saved board and sheet names. Find the board by name in Trello and the sheet "Life Automation Log" by title in Drive. Never use stored IDs. Keep IDs and dates as text when writing to the sheet (prefix with an apostrophe).

## Step 0b: Schedule check (Mondays only, scheduled runs only)
Skip on other days and on inline runs. On Monday: list_triggers (recurring, enabled; follow next_cursor) and find the task named "Life Automation: review". This task (named "Life Automation: prioritizer") must run exactly 75 minutes before it, same days. If its cron differs, update_trigger it (hour and minute computed minus 75, in the saved time zone) and carry on.

## Step 1: Read the board
Read the lists. Working lists are `TO DO` and `In Progress`. Ignore `Done` and `Proposed` everywhere in this skill. The three Snoozed lists are read for Step 1b and the Card index, and touched only in Step 1b. For each open card in the working lists and the Snoozed lists get: id, list, position, due date, description, last activity.

**Hidden cards (safety check).** The list reads can leave out a card whose position Trello cannot sort (it happened once: a card dropped into an empty list stayed invisible to every list read for a day, while the card itself and a search still showed it). In the same batch as the list reads, run a card search for `list:"In Progress"` and one for `list:"TO DO"` (trelloSearch `search_cards`; follow the pages while there are more). Any open card the search returns that the list reads did not return is a hidden card. Treat it as part of its working list for this run, and fix it by setting its position to the bottom of that list (trelloWriteCard, position only, nothing else), so list reads see it from then on. Count them for the final reply.

## Step 1b: Date-based moves (Trello writes)
"Today" and the weeks (Monday to Sunday) are in the saved time zone. Do (a) and then (b); the review used to do both itself, which made the start of every review slow.
**(a) Wake cards whose snooze date has arrived.** For each card in the three Snoozed lists whose metadata block has `Snoozed until` on or before today:
- Remove only these lines from the metadata block: `Snoozed until`, `Snoozed from`, `Wake on reply from`, `Watching since`, `Snooze type`, and any older `Woke on`. Add the header line `Woke on: YYYY-MM-DD (date reached)` (today's date) directly under `Last updated`; if the card had `Wake on reply from` and no `Snooze type: plain` (it was waiting on someone), write `Woke on: YYYY-MM-DD (date reached, waiting on <Name(s)>)` instead, with the names from the card's People lines that match those addresses (the addresses when no name is known). Write the rest of the description back exactly as it was, and leave `Last updated` unchanged. Never touch text outside the block.
- Move the card to the list named in `Snoozed from` (`TO DO` when the line is missing or that list does not exist).
- Add no comment: the `Woke on` line records the wake-up.
- Send the description update and the move as parallel calls, all cards in one message. A card whose `Snoozed until` is in the future stays where it is. Wakes caused by a reply are not done here: they need a live look at email, and the prep helper (or, when it did not run, the review) does them.
**(b) Roll the Snoozed lists forward.** Buckets: `Snoozed/Waiting this week` = `Snoozed until` on or before this Sunday; `Snoozed/Waiting next week` = next Monday through next Sunday; `Snoozed/Waiting later` = anything after next Sunday. Move every card that sits in the wrong list (for example on Monday everything in next week's list moves to this week's list), add the comment `**YYYY-MM-DD:** Moved to <list> (snoozed until YYYY-MM-DD)`, and do not change the metadata. Keep each Snoozed list sorted by `Snoozed until`, soonest at the top: put a moved card in its sorted place. If a Snoozed list is missing, skip the roll for it and say so in the run record (the review creates it).
**Failures.** Retry a failed write once, leave the card as it is and list it in the run record. The review does whatever is still open, so a failure here never loses a wake. Use the new lists and descriptions in Step 2 and Step 3.

## Step 2: Reorder within each working list
Woken cards are now part of their working list. Cards are only reordered inside a list here; the moves are the ones in Step 1b. Rules:
- **Primary: due date.** Overdue first (most overdue first), then soonest due, then cards with no due date.
- **Secondary, for cards without a due date:** recently active before dormant (smaller days since last activity first), then newer before older.
- Stable tie-break: existing order.
- Apply with trelloWriteCard (position only), only where the rank changes. Skip a list with fewer than 2 cards.

## Step 3: Refresh the Card index tab
The tab is a current-state snapshot, overwritten each run. Columns: `card_id, title, list, people_emails, thread_ids_hex, last_updated, refreshed_date, events, link_domains`.
- If the board read failed or returned no cards, leave the tab as it is and say so in the final reply.
- Otherwise compute all rows first, then batch_clear_values on `'Card index'!A2:I2000` and write one row per open card in `TO DO`, `In Progress` and the three Snoozed lists, from A2, in one update_values call.
- `card_id` = the card's object id. `title` = card title. `list` = list name.
- `people_emails` = the lowercase email addresses in the People lines of the card's `--- 🤖 Claude metadata ---` block, comma-separated (empty if no block).
- `thread_ids_hex` = the Gmail thread IDs in the block's Emails lines, as lowercase 16-character hex IDs, comma-separated, deduplicated. Gmail links come in several shapes: (a) URLs ending in `thread-f:<decimal id>`: convert the decimal to hex with `python3 -c "print(format(int('<decimal id>'),'x'))"`; (b) URLs whose fragment ends in a 16-character hex token (`#all/<hex>`, `#inbox/<hex>`, `#sent/<hex>`): use that token as is; (c) `thread-a:r<number>` URLs: ignore. Do the conversions in one python run, not by hand.
- `last_updated` = the block's `Last updated` date (empty if no block). `refreshed_date` = today in the saved time zone, YYYY-MM-DD.
- `events` = the date and time of every line in the block's `Events:` section that is not marked `(done)` or `(didn't happen)`, written only as `YYYY-MM-DD HH:MM` (24-hour; `YYYY-MM-DD` alone when the line has no time), separated by ` ;; `. No titles, names or places. Empty if none. Start the cell with an apostrophe.
- `link_domains` = the lowercase domains of the URLs in the block's `Others` and `Documents` lines, comma-separated and deduplicated, leaving out mail.google.com, drive.google.com, docs.google.com, calendar and maps links (google.com, maps.app.goo.gl), and trello.com. Empty if none. Example: a card whose Others line links `https://clinic.example.com/booking/...` gets `clinic.example.com`.

## Step 4: Final reply (the run record, 2-4 lines)
Cards woken (N), hidden cards fixed (N, with the card name and list), cards rolled between Snoozed lists (N), any move that failed, cards reordered per list, rows now in the Card index, whether the Monday schedule check ran and changed anything, any error. If nothing changed: "No reordering needed today." Do not email or message anyone.