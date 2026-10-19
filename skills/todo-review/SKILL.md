---
name: "todo-review"
description: "Daily walkthrough of the user's Life Automation Trello board. Use for the morning review, 'let's go through my tasks', or when /todo routes here."
---

# Daily Task Review (Life Automation)

A daily stand-up over the user's Trello board. First tell them where things are, then triage any tasks proposed from their Gmail, then go straight into the cards one by one (no asking first), suggest what to do, ask, and do it. Prep work for the cards runs in the background while they read the summary, and they can stop the review at any time.

In this skill "the user" is the person running the review. Write to them as "you".

## Flow at a glance
1. **Summary** (Step 2).
2. **Proposed tasks from Gmail** (if any) are triaged first, in quick batches (Step 2c). Prep for the regular cards runs in the background meanwhile.
3. **Card by card starts right after**, with no "ready to start?" question (Step 3).
4. **The user can stop at any time** (see **Stopping the review**).
5. **Wrap-up** with learnings (Step 5).

## Hard rule: never leave a card on your own
A card is finished only when the user chooses a finishing option in the check-in ("That's all, keep it in <list>", "Mark card as done" or the snooze until the meeting day, see Step 3.3), or when a Skip or Snooze button (or a suggested action that fully completes a single-part card) ended it, or when the user said to stop the review (see **Stopping the review**). If the user typed anything on a card, in "Other" or as a normal chat message, or asked a question, or reacted to a draft, finish with the check-in question (see Step 3.3) before showing the next card. This applies in every session and overrides any other step that says to go on to the next card.

## Before anything else: load settings
1. Read the memory file **`/areas/life-automation.md`**. It holds the user's settings: time zone, email lookback, home address (optional), plan type (scheduled or manual), board name, sheet name, and whether the daily schedule is paused.
2. **If the file is missing or has no board name, stop.** Tell the user in one line: "I can't find your Life Automation settings. Type /todo setup and I'll set things up." Do nothing else.
3. Use these values everywhere below:
   - **Time zone:** the saved zone. "Today", weeks and dates are always in that zone.
   - **Home address:** the saved address, or **none**. If none, see **Meetings and travel** (no travel events).
   - **Board:** the saved board name (default "Life Automation"). Find it by name with trelloSearch or trelloReadWorkspace. Never rely on a stored board ID.
   - **Log sheet:** the Google Sheet titled "Life Automation Log" (the saved sheet name). Find it by title with Google Drive search. Never rely on a stored file ID.
4. If the board cannot be found, or an expected list is missing, tell the user and point them to `/todo setup` (setup repairs a board, it never duplicates one).

## Setup facts (fixed for every user)
- Task list: the user's Life Automation Trello board. Trello is the ONLY task list.
- **The seven lists**, left to right: `Snoozed/Waiting later`, `Snoozed/Waiting next week`, `Snoozed/Waiting this week`, `TO DO`, `In Progress`, `Done`, `Proposed`. The names are exact. Never rename or translate them.
- **Working lists** are `TO DO` and `In Progress`. Nothing else is a working list.
- **Proposed list:** filled each morning by the discovery helper, which scans the user's Gmail for emails that need action and adds one card per email. Proposed cards are suggestions, not tasks yet: they are not part of the Step 0 order, are never snoozed, and are only handled in Step 2c. If the list is missing, create it at the right end of the board and say so in the brief.
- **Snoozed lists:** three lists, side by side and in this order: `Snoozed/Waiting this week`, `Snoozed/Waiting next week`, `Snoozed/Waiting later`. Every snoozed card lives in exactly one of them, picked by its `Snoozed until` date (see **Snoozed buckets**). "The Snoozed lists" in this skill means all three. If one is missing, create it next to the others and say so in the brief.
- Stale = no activity in 7+ days.
- Tools: Trello (cards and lists), Gmail, Google Calendar (optional), Google Drive and Sheets, scheduled tasks (paid plans only), Claude memory, a browser (optional, for public transport times).
- **Category labels** are whichever labels with a name exist on the board (for example Finance, Health, Family, Work). A new board has unnamed default labels, which are ignored; if there are no named labels the category check does nothing. This skill never creates labels, and the Trello connector cannot create them.

## Showing text (so it isn't collapsed)
In some places (e.g. the Chrome side panel), text written between tool calls gets collapsed into a one-line progress summary that the user can't read without clicking. So:

- **Everything the user needs to read** (the status brief, each card summary, each recap, email drafts) is sent with the **SendUserMessage** tool, which shows text exactly as written. If it's deferred, load it at the start of the run (ToolSearch `select:SendUserMessage`).
- Send it **right before** the button question it belongs to. The status brief is the exception: it is followed by background prep, not by a question (see Step 2b).
- If SendUserMessage isn't available, write the text as normal chat **and** repeat the card header plus the 1–2 status bullets at the top of the AskUserQuestion question text.

## Buttons (how to ask questions)
- **Every choice the user makes in this review is a clickable button**, asked with the **AskUserQuestion** tool. If that tool is listed as deferred, load it first (ToolSearch `select:AskUserQuestion`).
- **Never** write the options as a numbered list in chat and ask the user to reply with a number.
- Only if AskUserQuestion is truly unavailable, fall back to a short numbered list.

## Learnings (stored in Claude memory, not Trello)
What this review learns about how the user likes things done is kept in Claude memory, in the file **`/areas/life-automation-learnings.md`**.

- **At the start of every run,** read that file (memory_read) and follow everything in it. If it doesn't exist yet, carry on without it.
- **Learnings are things the user said or confirmed,** never guesses. Examples:
  - "Prefers Claude reminders, not Google Calendar events."
  - "Writes to German offices in German + English."
  - "Doesn't want health cards brought up unless asked."
  - "Usually picks 'nudge by email' over 'wait' for cards waiting 7+ days."
  - "Gmail discovery: don't propose emails from <sender>."
- **Gmail discovery learnings:** lines that start with `Gmail discovery:` are also read by the discovery helper to decide what NOT to propose. Keep that exact prefix on them.
- **How new learnings get added:** in Step 5, propose 0–2 learnings from today's session, e.g. when the user corrected a suggestion, rejected an action, picked something other than the recommended option, or dismissed several proposals of the same kind. Ask the user to confirm. Only save what they confirm.
- **If the user states a preference directly** during the review ("don't do X", "always do Y"), save it right away.
- **Saving:** read the file first, then memory_append one line per learning in the form `- [stated] <learning>`. If a new learning contradicts an old one, replace the old line (memory_str_replace) instead of adding a conflicting one. Create the file with memory_write if it doesn't exist, with frontmatter: name `life-automation-learnings`, a one-line description ("How the user likes the daily Trello task review run: confirmed preferences and learnings"), sources [cowork].
- Keep the file short. Merge similar lines rather than piling them up.
- Never store health details, finances or other sensitive personal details as learnings.

## Step 0: Prioritize cards
Whenever you go through cards in any step, order them like this:

The order is **the order the user's eyes follow on the Trello board**, so the walkthrough matches what they see on screen. Do NOT re-sort by due date, label or anything else.

1. **Lists, right to left.** Read the board's lists with `trelloReadList list_by_board` and sort them by `position`, ascending (left to right on screen). Keep only the working lists (`In Progress`, `TO DO`). Start at the working list **closest to `Done` on its left** (`In Progress`) and move left (`TO DO`).
2. **Within each list, top to bottom,** by the card's `position` (smallest first). Finish a list completely before moving to the next list on its left.
3. **Due dates do not change the order.** The prioritizer helper already puts due cards at the top of each list. Deadlines are surfaced in the ⏰ section of the brief and in each card's summary (see **Deadlines**), not by reordering.

Cards in the Snoozed lists are only handled under **Snoozing**. Cards in `Proposed` are only handled in Step 2c.

## Deadlines
- A card's **deadline** is the earliest of: its Trello due date, and any hard external deadline you found for it (a bill or statement due date, a filing deadline, an appointment). Say which one in the card summary, e.g. "due Mon Oct 12 (card) · statement due Oct 22".
- **A snooze must never hide a card past its deadline.** A snooze is only allowed if the card comes back **at least 1 day before** the deadline.
- Cards with a deadline in the next 7 days show up in **every** daily brief under ⏰ until they're done, unless the user snoozed them to a date before the deadline.

## Card metadata format
Metadata lives in the card's description field, inside this exact block:

```
--- 🤖 Claude metadata ---
Last updated: YYYY-MM-DD
Snoozed until: YYYY-MM-DD   (only present while the card is snoozed)
Snoozed from: <list name>   (only present while snoozed: the list the card returns to)
Wake on reply from: email@example.com   (only present with a "snooze until reply")
People:
- Name <email>, role (e.g. translator, accountant)
Documents:
- Title: Drive / Docs / Sheets / Slides / Forms / Claude artifact URL
Events:
- YYYY-MM-DD HH:MM: event title | calendar event URL   (a past event's line ends with (done), (not done) or (didn't happen) once its outcome is known)
Emails:
- Short description (who, when, what): Gmail thread URL
Others:
- Description: URL, or a plain note   (websites, Trello cards, Claude chats, videos, phone reminders)
--- end ---
```

Sections:
- **Fixed order:** People, Documents, Events, Emails, Others. A section with nothing in it is left out. Never write an empty header.
- **Put each item in the section by what the item is, not by where it came from.** Use judgment; it won't be perfect, and that is fine:
  - **People:** a person, or an organization, practice or service the user deals with, with email, phone and role.
  - **Documents:** anything stored as a file or document: Drive files and folders, Google Docs, Sheets, Slides and Forms, Claude artifacts. A document with no link yet is described in words.
  - **Events:** meetings, appointments, errands and calls that have a date and time. The event's calendar link goes at the end of the same line after ` | `, never on a line of its own. A calendar link with no known date goes on a line `Calendar event (date not recorded) | URL`. Sort by date, oldest first. A new in-person travel event is still not added (see **Meetings and travel**), but a travel event link already in a block stays, as `YYYY-MM-DD HH:MM: Travel: <meeting title> | URL`.
  - **Emails:** Gmail threads and messages. The helpers read this section to find a card's email threads (the card index is built from it), so every Gmail link goes here and nowhere else.
  - **Others:** everything else: websites, Trello card links, Claude chat links, videos, phone reminders, plans and notes without a type.
  - When an item could fit two sections, use the more specific one. When it is unclear, put it in Others. Never drop an item.
- **Instruction sections:** some cards carry their own section in the block, for example "Track in daily review (until the party):". Leave those exactly as they are, directly under the snooze lines and above People.
- **Keep each line's wording.** Sorting only moves lines to the right section.

Rules:
- **Only ever edit text inside this block.** Everything outside it is the user's own notes. Never change, reformat, move or delete it.
- When updating a description, read the current description first, replace only the block, and write back everything else exactly as it was.
- If the card has no block yet, add it at the **bottom** of the description, below the user's notes, separated by a blank line.
- If a description already contains a block, update it. Never add a second one.
- Update `Last updated` every time you change the block with real news or a real change. Step 1a uses it to know what counts as "new".
- `Snoozed until` in this block is the single source of truth for a snooze date. Never use the Trello due date for it: due dates are real deadlines.

## Proposal block (cards in the Proposed list)
The discovery helper writes this block into the description of every card it adds to `Proposed`:

```
--- 🤖 Claude proposal ---
Found: YYYY-MM-DD
From: Name <email>
Gmail thread: <URL>
Thread ID: <id>
Why: <one line: what is needed from the user, and by when>
Suggested list: <TO DO or In Progress>
Suggested due: <YYYY-MM-DD or none>
Possible match: <Trello card link or none>
--- end ---
```

Rules:
- The review only reads this block, until the user accepts the proposal in Step 2c. Never edit text outside it.
- When a proposal is accepted (Step 2c), replace the proposal block with the normal metadata block (`Last updated`, `People`, `Emails`). Never leave both blocks in one description.
- Do not set the Trello due date on a proposed card until the user accepts it.

## Meetings and travel
This applies **every time Claude creates a meeting (a Google Calendar event) as part of the review**, including errands, appointments, calls moved to a new slot, and anything else that lands on the calendar. It applies in every flow that uses this skill, including the single-card review. If Google Calendar is not connected, say so once, offer to note the meeting on the card instead, and skip everything below.

1. **Decide with AI whether the meeting is in person.** Read the title, description, location, any email thread and the card, and use judgment. In person: a physical address or venue, a clinic, office, pharmacy, notary, café or restaurant, "come to", "visit", an appointment at a place. Not in person: a video link (Meet, Zoom, Teams), a phone call, "online", "remote", or no sign of a physical place. If it's truly unclear, treat it as not in person and don't ask, unless the event is important enough that a missing travel block would be a real problem; then ask with a button.
2. **If it is NOT in person,** create only the meeting. No travel event.
3. **No home address saved:** create only the meeting, never a travel event. Mention once per review, in the recap, "No travel event, because no home address is saved (add one with /todo settings)." Items 4 to 8 below are skipped.
4. **If it IS in person and a home address is saved, also create a travel event** on Google Calendar:
   - **From address (always):** the saved home address.
   - **To address:** the meeting's address (the event location, or the address found in the email or card). Use the full street address when you have it.
   - **Travel time:** calculate it for **public transport** (bus, tram, metro, walking included) from the from address to the meeting address, for a trip that **arrives by the meeting's start time**. Use a browser (Google Maps directions in transit mode with "arrive by" set to the meeting time, or another reliable public transport route planner) and read the duration. Use the actual door-to-door time the planner shows. Don't add extra buffer on your own.
   - **If the meeting has no address,** or the route lookup fails, or no browser is available, default to **30 minutes** of travel time. A missing browser is never a reason to skip the travel event.
   - **Travel event:** title `Travel: <meeting title>`, ends exactly when the meeting starts, starts travel-time minutes before. Location = the meeting address (or blank if none). Description: from address, to address, the travel time and whether it came from the route planner or the 30-minute default, plus a one-line route summary (e.g. "Bus 280 → Metro M1") when you have it. Same calendar as the meeting. No guests, even if the meeting has guests.
   - If the travel window overlaps another event on the calendar, still create it and flag the overlap in the recap.
5. **Recap:** in the card recap (Step 3.5), list both, e.g. "Added meeting Thu Oct 15 14:00 (in person, Pharmacy X)" and "Added travel 13:25–14:00 (35 min by public transport)". For a default, say "30 min default, no address".
6. **Metadata:** add the meeting to the card's `Events:` list as usual, with the calendar event link at the end of its line. The travel event is not listed there, but if the meeting is later moved or deleted, update or delete its travel event too (and ask before deleting, as with stale meetings).
7. If the meeting is moved to a new time, recalculate the travel time for the new time and move the travel event with it.
8. **Check existing meetings for a missing travel event (card walkthrough only, never in unattended runs).** While re-checking a card's listed meetings against Google Calendar (Step 1a), look at each meeting that is in the future, within the next 14 days, and in person (same judgment as item 1). Use the calendar data you already fetched; do not scan the whole calendar, only the meetings listed on cards. A travel event is already there if an event whose title contains "travel" (e.g. `Travel: <title>` or just `travel`) ends at the meeting's start time or up to 15 minutes before it. If there is none, the card has an open item: show it as one bullet in the card summary ("No travel event for <meeting>, <date, place>") and put creating it (item 4 rules) in the suggested action's description, so it is also in the buttons. After it's created, carry on with the card's normal next step. If the user says they don't want a travel event for that meeting, add a Trello comment `**YYYY-MM-DD:** No travel event wanted for <meeting title> (<date>)` and don't ask again for that meeting. Never create it without the user's yes. This check is part of the card prep (Step 2b), so it runs in the background while the user reads the brief.

Outside item 8, this applies only to meetings Claude creates. Don't add travel events for meetings that already exist on the calendar unless the user asks.

## Meeting cleanup
Meeting cleanup covers **future meetings only**. **Never clean up, flag, list or recommend removing a meeting that is in the past**: past meetings stay in the card's `Events:` list as a record. A future meeting is **stale** when ANY of these is true:
- It no longer exists in Google Calendar. Before calling it stale, search the calendar for a moved or rescheduled version; if one exists, propose updating the entry instead of removing it.
- The card is snoozed until a date after the meeting, so the meeting is no longer what the card is waiting on.
- The user said it's done or irrelevant.

Rules:
- **Never remove a meeting without the user's yes.** Collect stale future meetings quietly in Step 1, across ALL cards including those in the Snoozed lists, then ask once in **Step 2a**.
- **Once confirmed,** remove only those lines from the block (nothing outside the block), update `Last updated`, and add one Trello comment per card: `**YYYY-MM-DD:** Removed meetings: <title> (<date>), ...`.
- **Unattended runs** (scheduled, nobody to answer): change nothing. List the stale future meetings in the brief so the user sees them next time.
- A stale meeting is never shown in a card summary as upcoming or pending, and never counts as news.
- If a stale meeting has a linked travel event (see **Meetings and travel**), include it in the same confirmation and remove it together with the meeting.

## Past meetings (follow-up check)
This applies in the card-by-card walkthrough and the single-card review, never in unattended runs. The lookup is read-only prep; the only thing that reaches the user is one question at the card.

- **What counts:** a meeting connected to the card that took place in the last 14 days (before today) and has **no recorded outcome**. Connected means it is listed in the card's `Events:` metadata, or it is a Google Calendar event that involves the people listed in the card's metadata or clearly matches the card's topic. A meeting has a recorded outcome when its line in `Events:` ends with (done), (not done) or (didn't happen). Use the calendar data you already fetched; don't scan the whole calendar.
- **When it runs:** when you reach a card (Step 3) that has such a meeting, before the card's normal question. The lookup is part of the card prep (Step 2b), so it's ready when the card is shown.
- **Show it in the summary:** one bullet in the card summary: "Past meeting: <title>, <date and time>".
- **Ask first, with its own question** (AskUserQuestion, before the card's normal action question): "Did the <title> meeting happen, and was the work from it done or resolved?"
  1. **"Yes, it happened and the work is done"**
  2. **"It happened, but the work isn't done yet"**
  3. **"It didn't happen"**
  The free-text "Other" box works as usual.
- **Act on the answer** (everything shows up in the card recap, and the card's normal question that follows is shaped by it). In every case, mark the outcome on the meeting's line in the card metadata, as `- YYYY-MM-DD HH:MM: <title> (done)`, `(not done)` or `(didn't happen)`, and update `Last updated`:
  - **Done:** add a dated Trello comment (`**YYYY-MM-DD:** Meeting <title> happened, work done`). Make updating the card the suggested action: tick the related checklist items (see **Stated done is the go-ahead**) and, if nothing is left, move it to `Done` (the checklist warning rule still applies).
  - **Happened, not done:** ask what is left (free text), add a dated comment with it, and make that remaining work the suggested action for the card.
  - **Didn't happen:** add a dated comment and make rescheduling the suggested action.
- Ask once per meeting. A meeting whose line is marked, or that the user already answered in this review, is never asked again.
- This check never removes the meeting from the card.

## Card content check
This applies in the card-by-card walkthrough and the single-card review, never in unattended runs. The check is read-only prep (Step 2b); changes need the user's yes.

- **Read the card's checklists directly** (trelloReadChecklist), because the plain card read can show none.
- **Compare** each checklist item, the title and the description with what the review found (emails, calendar, comments, activity, linked files). If developments made any of them outdated (an item that is done or changed, a title or description that no longer fits), show it as a bullet in the card summary and put the update in the suggested action's description, so it is also in the buttons.
- **Ask before changing.** Never leave them stale, and never change them silently.
- **Before marking any card done,** check its checklists. If items are unchecked, warn the user and ask before marking it done (a warning only, the user can override it).
- **Not wanted:** a checklist line in every recap, and a rule never to delete checklists.

## Stated done is the go-ahead
When the user says something is done (in "Other", as a chat message, or in an answer) and it clearly matches an unchecked checklist item, tick that item right away without asking, and list it in the recap. If the match is unclear, ask first with a button. This applies in the single-card review too.

## Snoozing
Snoozed cards live in the **Snoozed lists**. To snooze a card, always do all three:

0. If any of the three Snoozed lists is missing, create it with the exact name (`Snoozed/Waiting this week`, `Snoozed/Waiting next week`, `Snoozed/Waiting later`).
1. In the metadata block, set `Snoozed until: <date>` and `Snoozed from: <the list the card is in now>` (create the block with just those lines if the card has none yet, following the format rules). If the block lists any future meeting earlier than `Snoozed until`, flag it as stale for **Meeting cleanup** (ask; don't remove it silently).
2. Move the card to the right Snoozed list for its `Snoozed until` date (see **Snoozed buckets**).
3. Add a Trello comment `**YYYY-MM-DD:** Snoozed until YYYY-MM-DD` (for a snooze until reply: `Snoozed until <Name> replies or YYYY-MM-DD, whichever comes first`; for a snooze until the meeting: `Snoozed until YYYY-MM-DD (meeting day: <meeting title>)`).

Don't change the card's due date.

### Snooze fast path (any snooze button or typed snooze with a date the user chose)
A snooze is three plain writes, so do them in as few turns as possible:
- **Use what you already hold.** The card's description from the prep (re-read the card only if the cheap re-check in Step 2b showed it changed), and the list IDs from Step 1. Do not re-list the board, and do not re-read the card to "verify" before writing.
- **One parallel batch:** in a single turn send the three writes together: (1) update the description with the metadata lines (`Snoozed until`, `Snoozed from`, plus `Wake on reply from` for a snooze until reply, `Last updated` set to today), (2) move the card straight into the right Snoozed list (it goes directly from its current list; never via another list), (3) add the dated comment. They do not depend on each other.
- **No extra Step 3.4 round for a snooze or skip.** The metadata, move and comment above are the card update. Do not write the card a second time, and do not run the check-in for a snooze button.
- **Stale future meeting check** uses the `Events:` lines already in the description you hold (no new calendar search): a future meeting earlier than `Snoozed until` is flagged for **Meeting cleanup** as usual.

There are four kinds of snooze:

- **Plain snooze:** `Snoozed until: <date>`. The card stays hidden until that date.
- **Snooze until before the deadline:** `Snoozed until: <deadline minus 1 day>`. Used for cards with a deadline (see **Deadlines**).
- **Snooze until reply:** `Snoozed until: <follow-up date>` plus `Wake on reply from: <email>` (one or more addresses, comma-separated). The card stays hidden until **whichever comes first**: a reply from that person, or the follow-up date. `Snoozed from` is the list the card was in (`In Progress` or `TO DO`).
- **Snooze until the meeting day:** `Snoozed until: <date of the card's next future meeting>`. Used when the only thing still open on the card is an upcoming meeting (see Step 3.2 and Step 3.3). The card wakes in that day's review, before the meeting. The meeting stays in `Events:`: it is not earlier than `Snoozed until`, so it is not stale. Never offer it if the meeting day is later than 1 day before the card's deadline, and never when the meeting is today.

### The date is always the user's decision
- **Never choose a snooze date for the user, and never go on to the next card while a card being snoozed has no date the user chose.** A snooze is not done until the date is set.
- A button that shows its date in the label (e.g. "Snooze 7 days (Thu Oct 15)") counts as the user choosing that date.
- If a snooze is needed and the user hasn't picked a date, ask with AskUserQuestion: "Snooze [card] until when?" This happens when:
  - the action leaves the ball in someone else's court (e.g. we just sent an email, so a snooze until reply is needed),
  - the user chooses "Other" and says "snooze it" or similar without a date,
  - the user gives a vague date ("next week").
- Options for that question: **7 days (Recommended when waiting on someone)**, 3 days, 14 days, each with its real date in the label. Drop any date that would come back later than 1 day before the card's deadline. The free-text "Other" box lets the user type a specific date.
- If the user types a date that's too late for the deadline, say so and ask again. Stay on the card until a valid date is set.
- "Skip for today" is not a snooze: it changes nothing and needs no date.

### Waking cards
- **Step 1 reads all three Snoozed lists.** For each card in them:
  - If it has `Wake on reply from`, search Gmail for messages from those addresses since the snooze was set. If one arrived, **wake the card**: remove the snooze lines, move it to `In Progress`, and list it under 🆕 in the brief ("Reply from X").
  - If `Snoozed until` is today or earlier, wake it: remove the snooze lines and move it back to the list in `Snoozed from` (if that's missing, use `TO DO`). If it was a snooze until reply with no reply, bring it back with the follow-up as the suggested action (e.g. "Nudge <person> by email").
- Cards in the Snoozed lists are skipped in Step 3 and left out of the Step 2 brief, except: show them if they have a Trello due date on or before today, or if a new email or reply arrived for them (a snooze never hides a real deadline or real news).
- **Migration (any time you find them):** a card with a `Snoozed until` line that is not in a Snoozed list gets moved to the right one, and `Snoozed from: <its current list>` is added to its metadata. Mention it in the brief once.
- In Step 2, add one line: "😴 Snoozed: N this week · N next week · N later" (no details).

### Snoozed buckets
Weeks run Monday to Sunday (in the user's time zone). Compare the card's `Snoozed until` date with today:
- **`Snoozed/Waiting this week`**: on or before this Sunday.
- **`Snoozed/Waiting next week`**: from next Monday through next Sunday.
- **`Snoozed/Waiting later`**: anything after next Sunday.

**Rolling forward (every run, in Step 1, after waking cards):** check each card in the Snoozed lists against these rules and move any card that's in the wrong bucket (for example, on Monday everything in next week's list moves to this week's list, and cards in the later list move up once their date falls inside next week). Add a comment on each moved card: `**YYYY-MM-DD:** Moved to <list> (snoozed until YYYY-MM-DD)`. Don't change the metadata, and don't mention routine moves in the brief. Within each list, keep cards sorted by `Snoozed until`, soonest at the top.

A card moving between Snoozed lists is still snoozed; `Snoozed from` keeps pointing to the list it returns to when it wakes.

## Category labels
Every card should carry a category label (one of the board's existing labels, e.g. Finance, Health, Family, Work). Claude checks this on **every review run**, so a card never stays unlabeled by accident. Snoozed cards are not shown in Step 3, so the sweep in Step 1 is the only place they get checked.

### Which cards
Every open card with **no label** in any list except `Done` and `Proposed`, **including the three Snoozed lists**, unless it has a Trello comment containing "No labels wanted". Cards that already have a label are never touched: this check adds labels, it never changes or removes them.

### Deciding
- Read the board's labels (trelloReadBoard `list_labels`). Ignore labels with no name. **Never create a new label.**
- Judge each card from its title, description, metadata (People roles, Events), comments and the learnings file. Same bar for every label.
- **Sure:** the card's subject clearly falls inside one label's meaning (for example a doctor, clinic or medical appointment fits Health). A card gets two labels only if both are clearly right.
- **Not sure:** several labels could fit, none fits well, or the card is too vague. Then it is not sure; never guess.

### Sure: attach it, no question
When Claude is sure, it attaches the label itself (trelloWriteCard `attach_label`), without asking. This runs in Step 1 of every run, interactive or unattended. Report each one as a bullet under 🆕 in the brief ("Added label Health to [card](link)") and in the Step 5 wrap-up.

### Not sure: suggest it, apply only on the user's yes
- Only for a card that is shown in Step 3 (or in the single-card review). Snoozed or unattended cards that are not sure are left alone until they come up.
- Suggest one or more of the existing labels with a one-line reason, as a bullet in the card summary ("No category label. Suggest: Health (doctor appointment)") and in the suggested action's description, so it is also in the buttons. Choosing the suggested action counts as the user's yes; apply the label then.
- If the user says no to the label (button text or typed), add the Trello comment `**YYYY-MM-DD:** No labels wanted` and never suggest again for that card.
- Never suggest for a card that already has a label.
- Mention a label added this way in the card recap.

## Linked Google files (check for news)
Applies to every card, with or without metadata, that has a Google Sheets, Docs or Slides file linked: in its attachments, description, comments or the metadata `Documents:` section (docs.google.com or drive.google.com URLs). It is read-only prep and runs in interactive and unattended runs. It exists because a comment on a linked guest-list sheet ("she said she can't make it") can be missed by a review that only reads the card.

- **Reference point for "new":** the metadata `Last updated` date. If the card has no metadata, use the date of the latest dated Claude comment on the card, and if there is none, the card's creation date.
- **Always read with comments turned on.** Without the comments option the connectors return only cell or text content, never comments.
  - Sheets: Google Sheets `get_spreadsheet` with `commentsIncluded: true`. It returns each thread's author, create and update times, OPEN or RESOLVED status and the anchored cell. For the cell content and the quoted text of each comment, also use Google Drive `read_file_content` with `includeComments: true`.
  - Docs and Slides: Google Drive `read_file_content` with `includeComments: true`.
- **Don't use the file's modified time as a shortcut.** Comments may not change it, so always fetch the comments.
- **What counts as new:** a comment thread or reply created or updated after the reference point, and a thread resolved or reopened since then.
- **Show each new comment with its context:** the commented text or cell and the row it belongs to. A bare comment is not enough to act on.
- **Also compare status-style columns or fields** (e.g. confirmed, RSVP, status, done) with what the card's latest comment and description say. If a value changed since the card last reflected it, report it as news.
- **Leave a snapshot for next time:** when the review writes a comment on a card that has such a sheet, add a one-line summary of the key counts (e.g. "RSVP: 2 yes, 1 can't make it, 12 no reply") so the next review can compare against it.
- **Treat new comments and changed statuses as news:** count them in the 🆕 line of the brief and show them as the first bullet of the card summary. What to do about them (update the card, its checklist or title, reschedule, tell someone) goes into the suggested action or the buttons.
- **If a linked file can't be read** (no access, connector error), say so in the card summary ("Couldn't read comments on linked <file name>, please check manually"). Never skip it silently.
- Nothing is changed in the Google file, and nothing on the card changes without the user's yes.

## Review prep cache (the morning prep, paid plans)
The scheduled prep helper does the slow read-only prep ahead of time and writes it to the **Review prep** tab of the Google Sheet "Life Automation Log". Use it, so the cards open fast. On non-paid plans there is no prep helper, so the cache is never there: prep live, with no warning line.

- **Columns:** item_type (meta/card/proposal), item_id (card object id), list, deadline, new_thread_ids_hex, calendar_change (none/new/moved/deleted), news, suggested_update, suggested_action, proposal_suggestion, merge_target_card_id, reason_codes, confidence, prepared_at (UTC), prepared_date (user's date), notes. It holds only pointers and codes, never free text, so every summary is still written fresh from the card itself.
- **When it is usable:** read the tab once in Step 1 (Google Sheets get_values, in parallel with the other Step 1 reads). Row 2 is the meta row: the cache is usable only if its `prepared_date` is today (user's time zone) and its status (in the `news` column) is `ok` or `partial`. Otherwise (paid plans only) say once, in one line in the brief, "No prep today (<reason>), prepping live", and prep live as described in Step 2b. Never fail or wait because of it.
- **Trust rules, per card:** trust a card row (find it by `item_id` = the card's object id) unless one of these is true; then prep that card live: confidence `low`, suggested_action `needs_review`, reason code `no_prep_cap`, the card has `Wake on reply from`, the card has Trello activity later than `prepared_at`, or the card has no row. A card that contradicts its row always wins over the row.
- **What a trusted row saves:** skip the Gmail, Calendar and Drive searching for it (the row's `new_thread_ids_hex`, `calendar_change`, `reason_codes` and `suggested_update` already say what changed). Still read the card itself (description, checklists, comments), and still open each thread listed in `new_thread_ids_hex` by ID with get_thread.
- **One freshness check for the whole review:** the cache was written before the review, so after reading it run ONE inbox-wide Gmail search for threads newer than the meta row's `prepared_at`, and match the results against the Card index `people_emails`. Any hit marks that card "live prep" (its row is not trusted for email news). Repeat this single search at most once more in the review, never per card. Calendar changes after `prepared_at` are caught by the card's own meeting re-check (Meeting cleanup), not by a new search.
- **Using the row's suggestions:** `suggested_action` picks the Recommended button for the card (send_email, snooze_until_reply, snooze_until_meeting, schedule_meeting, update_card, mark_done, nudge; none means keep it open, needs_review means decide yourself). The row never decides a snooze date: dates come from the rules in **Snoozing** and the buttons. `reason_codes` and `suggested_update` drive the summary bullets (for example `missing_travel`, `meeting_past_no_outcome`, `checklist_outdated`, `deadline_soon`, `waiting_7d`). If the card shows the suggestion is wrong, say so briefly and use your own.
- **Proposals:** `proposal_suggestion` (add, merge, dismiss, completed) picks the Recommended button in Step 2c; `merge_target_card_id` is the card to merge into.
- The review never writes to the Review prep tab.

## Step 1: Gather (quietly, no narration)
Keep this phase lean: do only what the Step 2 brief needs, so the user gets the summary fast. Per-card deep work (enrichment, travel checks, drafts) is prep and happens after the brief is sent (see Step 2b).

- Load the settings (see **Before anything else**), read the learnings file (see **Learnings**), and, in the same batch of parallel calls, the **Review prep** tab (see **Review prep cache**).
- Read cards in `In Progress` and `TO DO`, with due dates, checklists, labels and the latest comments.
- Read the `Proposed` list (cards with their proposal blocks). They stay out of the regular card order and out of the Snoozed handling. Count them for the brief, and note how many were found more than 7 days ago.
- Read the three Snoozed lists, wake cards as needed and roll the rest forward (see **Snoozing** and **Snoozed buckets**). Create any missing list.
- Cards still in a Snoozed list stay out of the rest of the review, **but still get a Meeting cleanup check** (see **Meeting cleanup**): that's where stale meetings pile up.
- For every card with an `Events:` list, check each future entry against **Meeting cleanup** and collect the stale ones for Step 2a. Ignore past meetings here (they are handled by **Past meetings** at the card).
- Work out each card's deadline (see **Deadlines**).
- Run the **Category labels** check on every open card without a label, **Snoozed cards included**: attach the label where Claude is sure, and keep the not-sure ones for their card summary. Count the added labels for the 🆕 section of the brief.
- Go through the rest in the Step 0 priority order.

### 1a. Cards that already have metadata (check for news)
- If the **Review prep cache** is usable and the card's row is trusted, take the news from the row (and open its listed threads by ID) instead of searching; the rest of this section applies to cards that need live prep.
- Only look for **new** emails and calendar events involving the people already listed in the card's metadata, since the block's `Last updated` date.
- **Email news comes from the Card news tab first** (when the sheet has one; it is filled by the discovery helper). Read the **Card news** tab of the Google Sheet "Life Automation Log". It holds every thread that involves a person on an open card (columns: found_date, thread_id, last_msg_date, card_id, sender_domain, category, needs_action, notes). Take the rows for this card's `card_id` whose `last_msg_date` is later than the block's `Last updated`, and open each thread with get_thread (thread_id is Gmail's hex ID). That is the card's email news. Run your own Gmail search for the card's people only when: the card is not in the **Card index** tab of the same sheet (new, or excluded), its Card index `refreshed_date` is more than 2 days old, there is no Card news tab or it is empty (non-paid plans before the first helper run), or the card has `Wake on reply from` (see **Snoozing**). Calendar checks are unchanged.
- Don't re-search old history, **except** re-check each future meeting already listed against Google Calendar (see **Meeting cleanup**). In the card walkthrough, also check each upcoming in-person meeting for a missing travel event (see **Meetings and travel**, item 8).
- For cards with a linked Google Sheets, Docs or Slides file, also run the **Linked Google files** check (new comments, changed statuses) since `Last updated`.
- Feed anything new into the Step 2 status brief.

### 1b. Cards without metadata (enrich gradually)
- Enrich at most **3 cards per day**, picked in the Step 0 priority order. The board fills in over time.
- Do the enrichment as part of card prep (Step 2b), after the brief has been sent, not before it.
- Skip cards with too little info to search on (a vague title, no comments). Don't "burn" tokens on them; focus on easier matches.
- For each of the 3 cards, try to find:
  - Who the card is connected to. Look at emails in Gmail and meetings in the calendar. Try to understand the meaning of the card and look for related emails and messages, e.g. if the card is about taxes, find emails about taxes.
  - What meetings are connected to the card.
- **Search window for this first enrichment only:** emails and calendar events within 30 days of the card's creation, i.e. 30 days before and after. After that, the card follows 1a and only new activity since `Last updated` is checked.
- Keep the findings for Step 3. **Write the metadata right away** when the card is shown in Step 3 (prep stays read-only) and say what was added in the card summary, instead of asking for confirmation first. If nobody is there to answer (e.g. a scheduled run), write nothing.
- Write the metadata into the card's description using the **Card metadata format** above, with every item sorted into its section.

## Step 2: Status brief (short)
Send the brief with SendUserMessage (see **Showing text**).

Sections, in this order. Use only these icons, no new ones. Plain text only.

- 🆕 **What moved since last time:** new replies (including woken cards), changed cards, labels added by the Category labels check.
- 📥 **Proposed:** N proposed tasks from Gmail waiting for triage (K found more than 7 days ago). Only if the `Proposed` list has cards.
- ⏰ **Due soon:** overdue, due today, and due in the next 7 days (these repeat every day until done).
- ⏳ **Waiting on:** who, since when, follow-up due (cards snoozed until reply).
- 💤 **Stale.**
- 😴 **Snoozed:** N this week · N next week · N later.
- 🧹 **Stale future meetings:** N on M cards (only if any were found; unattended runs list them here instead of asking).

**Layout (one item per line, never bundle several items into one line):**
- A section with several items gets a header line (icon + short title), then **one item per line** as a bullet (`- `) directly underneath. Put a blank line between sections.
- A section with nothing to show collapses to a single line: `💤 Stale: none`, `🆕 Nothing new since the last run`.
- 😴 Snoozed, 💤 Stale (when none) and 🧹 stay single lines. The ▶️ line stays last.
- **⏰ Due soon:** date first, then the card link and a short detail in brackets. Sort by deadline, today first. Format: `- Today: [card](link) (dentist 10:30)`, `- Fri Oct 16: [card](link) (wakes that day)`. When a snooze date and a deadline coincide, say so in the brackets.
- **⏳ Waiting on:** `- Who, [card](link): since <date>, follow-up <date>` (or `last reply <date>` when they replied).
- **🆕:** one bullet per news item (a reply, a woken card, a card moved, a changed card, a label added), each with its link.
- Cap each section at about 5 bullets. If there are more, show the 5 most urgent and end with `- and K more`.
- Every bullet is one short line with a link. No long prose. **Don't show any individual card details or options in the brief.**
- **One-time notes** go as a single bullet under 🆕 when they apply: "No home address saved, so I'm not adding travel events (add one with /todo settings)", and, on paid plans, "No prep today (<reason>), prepping live".

Example:

```
Daily brief, Wed Oct 14

🆕 Nothing new since the last run
- No wake-ups, no replies on snoozed cards

⏰ Due soon
- Today: [fix the toothpain](link) (dentist 10:30)
- Fri Oct 16: [birthday party](link)
- Mon Oct 19: [stop subscription payment](link) (snoozed until then)

⏳ Waiting on
- Dana: since Oct 10, follow-up Oct 19
- Alex, [car seat](link): follow-up Oct 15

💤 Stale: none
😴 Snoozed: 8 this week · 4 next week · 6 later

▶️ Starting the card-by-card review now (13 cards). Say "stop" anytime to end it.
```

End the brief with one last line (interactive runs only). With proposals: "▶️ Starting now: M proposed tasks from Gmail, then N cards. Say 'stop' anytime to end it." Without proposals: "▶️ Starting the card-by-card review now (N cards). Say 'stop' anytime to end it." Use the real counts.

## Step 2a: Confirm meeting cleanup
Only if Step 1 found stale future meetings (see **Meeting cleanup**) and the user is present. Launch the card prep first (Step 2b, item 2) so it runs while the user answers. Then send one SendUserMessage listing each affected card with its stale future meetings and why (deleted from calendar, before snooze date, or the user called it irrelevant), then ask **one** question with AskUserQuestion for all of them together:

- Question: "Remove these N stale meetings from M cards?"
- Options:
  1. **"Yes, remove them (Recommended)"** (description: "Removes the lines from card metadata and logs a comment on each card")
  2. **"Keep them for now"** (description: "No changes; I'll ask again next review")
- If the user types something in "Other" (e.g. "keep the Dec 3 one"), follow it exactly.

Do the removals as described under **Meeting cleanup**, then continue to Step 2c (or Step 3 if there are no proposals), with no extra question. Include them in the Step 5 wrap-up.

## Step 2b: Start right away, and prep while the user reads
**Do not ask whether the user is ready.** Right after the brief, the review starts on its own: first the proposed tasks (Step 2c, if any), then the cards (Step 3). The goal: while the user reads the summary and triages proposals, you already do the slow work for the cards, so each card is ready to show the moment they get to it.

1. Send the brief (Step 2), ending with the "Starting" line.
2. **Immediately after sending the brief** (before Step 2a's question if there is one, and without waiting), start the prep for the cards in Step 0 order, in parallel. Prep for card 1 is the priority; show card 1 as soon as its prep is done, without waiting for the other cards.
3. **Where the cache is usable (see Review prep cache), a trusted card needs only a light prep:** read the card itself and its checklists, open the threads listed in its row, and build the summary and buttons from the row's `suggested_action`, `suggested_update` and `reason_codes`. The list below is the full live prep, used for cards without a trusted row or when there is no cache today.
   **Prep per card is read-only and covers:**
   - the full card (description, checklists read directly, comments) and its deadline,
   - news since `Last updated` for the people in its metadata (Step 1a),
   - new comments and changed statuses in any linked Google Sheets, Docs or Slides file (**Linked Google files**),
   - for cards without metadata, the enrichment findings (Step 1b, still capped at 3 per day),
   - the missing-travel-event check for upcoming in-person meetings (**Meetings and travel**, item 8),
   - the lookup of past meetings with no recorded outcome (**Past meetings**),
   - the **Card content check** (checklist items, title and description against what was found),
   - whether the only thing still open on the card is an upcoming meeting (see Step 3.2), so the snooze until the meeting day can be the suggested action,
   - the label suggestion for a card the Category labels check was not sure about,
   - the suggested action, with the option descriptions already worked out (Step 3.2),
   - any email draft the suggested action needs (drafted, never sent),
   - the public-transport travel time for a suggested in-person meeting (only if a home address is saved),
   - the card summary bullets (Step 3.1), ready to send.
4. **How to run it in parallel:** if the Agent tool is available, use one subagent per card (or small batches) that returns a ready-to-use prep note; otherwise batch independent tool calls in a single message. Don't prep every card on the board up front. Keep roughly the next 3 cards prepped: while the user is answering card N, prep for N+1 and N+2 is running or done, and you start prep for the next card as each one is shown. The proposed tasks need no prep: the discovery helper already wrote what's needed into each card.
5. **Prep never changes anything.** No Trello writes, no emails sent, no calendar events created, no learnings saved. Everything still needs the user's click or yes at the card, exactly as in Step 3, Step 4 and Step 5. (The sure labels from the Category labels check were already attached in Step 1.)
6. **Before showing a prepped card,** re-check that it hasn't changed since prep with ONE cheap call (a single card read for its list, latest comment and `Last updated`), batched in the same turn as any other call you are making for that card. Do not re-verify anything else, and do not re-read lists. If something changed, or something the user just did on an earlier card affects this one, redo only the affected part first.
7. Unattended runs (scheduled, nobody to answer) have no card-by-card part, so no prep either.

Then go to Step 2c with the first proposal, or to Step 3 with card 1 if there are no proposals.

## Step 2c: Triage proposed tasks from Gmail
Only in interactive runs, and only if the `Proposed` list has cards. It starts right after the brief (and Step 2a), with no extra question. Prep for the regular cards is already running in the background (Step 2b).

- **Order:** soonest `Suggested due` first (no due date last), then oldest `Found` first.
- **Show, then ask, in batches of up to 4 proposals.** For each batch, send ONE SendUserMessage listing the proposals: `**Proposed X of M:** [title](card link) · from <sender> · <Why> · suggested <list>, due <date>`, with the Gmail thread link. Then ask with ONE AskUserQuestion call with one question per proposal (header = a few words of the title). Repeat for the next batch until all are done.
- **Which option is Recommended:** "Add" by default. If the **Review prep cache** is usable and has a row for the proposal (`item_id` = its card object id), follow its `proposal_suggestion` with the "(Recommended)" tag: add, merge (into `merge_target_card_id`), dismiss or completed (= "Already completed"). Ignore a row with confidence `low`. Move the "(Recommended)" tag to that option and keep the order below. The proposal block and the Gmail thread win if they contradict the row.
- **Options for each proposal, in this order** (drop "Merge" if `Possible match` is none):
  1. **"Add to <list> (Recommended)"**. Description: "Moves the card to <list>, sets due <date> (if any) and links the Gmail thread". Use the `Suggested list` and `Suggested due` from the proposal block.
  2. **"Merge into <card name>"**. Description: "Adds the email to that card (comment and link) and archives this proposal".
  3. **"Dismiss"**. Description: "Archives the proposal and I'll avoid similar ones".
  4. **"Already completed"**. Description: "Archives the proposal. It was a real task and you already handled it, so I'll still propose similar ones".
- The free-text "Other" box is the user's own instruction for that proposal (e.g. "add to In Progress, due Friday"). Do what they asked. If they wrote anything themselves, send a short recap and ask the check-in question ("Anything else on these, or move on?") before the next batch or Step 3, as in the Hard rule.
- **Act on the answers:**
  - **Add:** move the card from `Proposed` to the chosen list. Replace the proposal block with the normal metadata block (`Last updated`, `People` with the sender, `Emails` with the Gmail thread). Set the Trello due date to `Suggested due` only because the user chose that button. Keep the title unless they typed a change. Add a comment `**YYYY-MM-DD:** Added from Gmail: <subject> (<thread link>)`. Leave the position to the prioritizer.
  - **Merge:** add a comment on the target card with the thread link and a one-line summary. Add the thread to its `Emails` and the sender to its `People` (metadata rules apply, update `Last updated`). Then archive the proposal with a comment `**YYYY-MM-DD:** Merged into <card name>`.
  - **Dismiss:** add a comment `**YYYY-MM-DD:** Dismissed (<sender>: <subject>)` and archive the card. If the user dismissed 2 or more proposals of the same sender or kind in this review, propose a `Gmail discovery:` learning in Step 5.
  - **Already completed:** add a comment `**YYYY-MM-DD:** Already completed (<sender>: <subject>)` and archive the card. The comment must start with the words "Already completed": the discovery helper reads it to tell this apart from a dismissal. It is not a dismissal: never count it toward a `Gmail discovery:` learning, never propose one because of it, and keep the card out of the Dismiss tally.
  - **Skip in the question UI, or no answer:** change nothing. The proposal stays in `Proposed` and comes back in tomorrow's review. There is no "leave for later" button, because a proposal is either a task (Add, Merge), not a task (Dismiss) or already handled (Already completed).
- Keep a tally for the Step 5 wrap-up (added, merged, dismissed, already completed, skipped).
- **Unattended runs:** no triage. Only count the proposals in the brief.
- When triage ends, go to Step 3 with card 1.

## Step 3: Card by card
Take the user through the cards **one at a time**, in the Step 0 priority order. Proposed tasks are not part of this: they were handled in Step 2c, and "N" below counts only the regular cards. Never show the next card until the current one is answered and handled (and, if the user is snoozing it, until its snooze date is set). For Snooze and Skip answers, "handled" means the writes have been sent: the next card can go out while they finish (see **Don't wait for the writes** in 3.5). Thanks to the prep in Step 2b, each card should usually be ready to show right away; if its prep isn't done yet, finish it and show the card. For each card:

1. **Card header and summary, always shown before the question.** Send it with SendUserMessage (see **Showing text**), in this shape:
   - The recap of the previous card first (see 3.5), if there was one.
   - `**Card X of N:** [card name](link) · list · deadline if any`
   - 1–3 bullets on where it stands: latest comment, new emails (with thread link), what it's waiting on, money or dates involved, the user's own next step. Only mention meetings that are still in the future as upcoming; a past meeting is only shown through the **Past meetings** follow-up. If an upcoming in-person meeting has no travel event, add one bullet for it (see **Meetings and travel**, item 8).
   - If the **Card content check** found an outdated checklist item, title or description, one bullet for it.
   - If the card has no label and the Category labels check was not sure, one bullet with the label suggestion (see **Category labels**).
   - If Step 1b found metadata for this card, one bullet saying what metadata was written.
   - No numbered options in the text. The options are the buttons in the next step.
   - **Never skip the summary**, even when the card seems obvious.
2. **Ask one question for this card** with AskUserQuestion. One card per question; never batch several cards into one question. Pick the options like this. Every snooze option shows its real date in the label:

   **Cards with no deadline, or a deadline more than 31 days away:** exactly these four, in this order:
   1. **The suggested action**, labeled with the action and "(Recommended)". If the card has a trusted row in the **Review prep cache**, start from its `suggested_action`. Put what it will do in the option's description (including any checklist, title or description update from the Card content check, adding the suggested label, if any, and, for a suggested meeting, the travel event that goes with it if it's in person, and creating any missing travel event named in the summary). Shape the suggestion using the saved learnings and the card's emails, calendar and activity.
      - **When the card is waiting on someone** (we already emailed or asked them and nothing has come back), the suggested action is a snooze until reply, with a **7-day** follow-up, worded with "Snooze" so it matches the other snooze buttons: **"Snooze until <Name> replies or <Day Mon D> (Recommended)"** where the date is today + 7, e.g. "Snooze until Dana replies or Thu Oct 15 (Recommended)". Description: "Hidden in Snoozed until they reply or Oct 15, whichever comes first. If there's no reply by then, it comes back with a nudge." If the card has a deadline, the follow-up date must be at least 1 day before it; use the earlier date.
      - **When the only thing still open on the card is an upcoming meeting** (a future, not stale meeting in `Events:`, no unchecked checklist items, no reply awaited, no email to send, no missing travel event), the suggested action is a snooze until the meeting day: **"Snooze until <Day Mon D> (meeting day) (Recommended)"**, where the date is the earliest such meeting's date. Description: "Hidden in Snoozed until the day of the meeting, then back in your daily review." Skip this if the meeting is today, or if its date is later than 1 day before the card's deadline. The date in the label counts as the user's choice.
   2. **"Snooze 7 days (<Day Mon D>)"**
   3. **"Snooze 30 days (<Day Mon D>)"**
   4. **"Skip for today"**

   **Cards with a deadline in the next 31 days:** never offer a snooze that would bring the card back later than 1 day before the deadline. Options, in this order (drop any that don't apply):
   1. **The suggested action (Recommended)**, as above. If it doesn't finish the task, the description ends with **"Stays in your daily review until it's done."**
   2. **"Snooze until <Day Mon D>"**: the day before the deadline. Description: "Hidden in Snoozed until the day before it's due, then back in your daily review." Only offer it if that date is at least 2 days from today.
   3. **"Snooze 7 days (<Day Mon D>)"**, only if today + 7 is at least 1 day before the deadline and different from option 2.
   4. **"Skip for today"**: description "No changes; it's back in tomorrow's review (due <date>)."
   - Never show **"Snooze 30 days"** for these cards.

   The tool adds a free-text **"Other"** box automatically. That's the user's own action or a chat about this card. It is also where they can type "stop" (see **Stopping the review**).
3. **Act on the answer:**
   - **Suggested action:** do it (see Step 4; emails still need the user's yes on the final text), including attaching the suggested label if the description named one. If the action creates a meeting, follow **Meetings and travel**. If the task isn't finished, leave the card unsnoozed so it shows up in every review until it's done. If the action leaves the ball in someone else's court, a snooze until reply is needed: ask for the date (see **The date is always the user's decision**). If the suggested action is the snooze until the meeting day, follow **Snoozing**.
   - **Snooze until reply:** follow **Snoozing** (metadata with `Snoozed until`, `Snoozed from` set to the card's current list, and `Wake on reply from`, add the person to People if missing, move to the right Snoozed list, comment).
   - **Snooze until before the deadline / Snooze 7 / 30 days / Snooze until the meeting day:** follow **Snoozing** (metadata, move to the right Snoozed list, comment). Don't change the card's due date.
   - **Skip for today:** change nothing on the card. Just move on; it comes back in tomorrow's review as usual.
   - **Stop / that's enough / I'm out of time (typed in "Other" or as a chat message):** follow **Stopping the review**.
   - **Other (the user's text):** treat it as a conversation about this card. Do what they asked, ask follow-up questions if needed, and stay on this card until it's handled. A normal chat message typed on this card counts the same as "Other". If they say something is done, apply **Stated done is the go-ahead**. If they decline the suggested label, add the "No labels wanted" comment (see **Category labels**). Then do the **check-in** below. Do NOT move to the next card on your own.

   **Check-in (whenever the user wrote anything themselves, by "Other" or as a normal chat message, or the card still has parts left):**
   - After doing what was asked (and the Step 3.4 card update), send a short recap of what was done (SendUserMessage, same shape as the Step 3.5 recap), then ask one question with AskUserQuestion. The question's first sentence says what was done and where the card now sits, so the user never thinks the card changed state when it didn't: "<what was done>. The card stays in <list>. Anything else on it?" (for example "Thank-you email sent and logged. The card stays in TO DO. Anything else on it?"). `<list>` is the card's real list after the Step 3.4 update. If the card moved (to a Snoozed list), say so instead of "stays": "The card is now in <list>."
   - **Work out first whether the only thing still open on the card is an upcoming meeting** (the same test as in Step 3.2: a future, not stale meeting in `Events:`, no unchecked checklist items, no reply awaited, no email to send, no missing travel event; and the meeting is not today, and its date is not later than 1 day before the card's deadline). Then the options are:
     1. **"Snooze until <Day Mon D> (meeting day) (Recommended)"** (description: "Hidden in Snoozed until the day of the meeting, then back in your daily review."). The date is the earliest such meeting's date. The date in the label counts as the user's choice.
     2. **"That's all, keep it in <list>"** (description: "Card stays open in <list>. Moves on to the next card."). Not Recommended in this case.
     3. **"Something else on this card"** (description: "I'll tell you what to do next.").
     - **"Mark card as done" is never shown in this case**, because a meeting is still ahead.
     - If the user chooses the snooze: follow **Snoozing** (metadata, move to the right Snoozed list, comment), mention it in the recap, and go on to the next card. It is a Snooze button, so no further check-in.
   - **In every other case** (anything else is still open, or there is no upcoming meeting), the options are, in this order:
     1. **"That's all, keep it in <list> (Recommended)"** (description: "Card stays open in <list>. Moves on to the next card."). Recommended whenever the card is still open work.
     2. **"Mark card as done"** (description: "Moves it to the Done list (I'll check its checklist first)."). Only show it when the work looks finished: nothing left open, no unchecked checklist items, no reply being waited on. If it isn't clearly finished, leave this option out. When it is clearly finished, make this the Recommended option instead of the first one.
     3. **"Something else on this card"** (description: "I'll tell you what to do next.").
   - **Never label a button plain "Done" or "Finish"** unless choosing it moves the card to the Done list. The wording must always tell the user what happens to the card.
   - The free-text "Other" box also works for typing the next ask directly (including "stop").
   - If the user chooses "Mark card as done": run the checklist check first (see **Card content check**: if items are unchecked, warn and ask before moving it), then move the card to `Done` and add a dated Trello comment (`**YYYY-MM-DD:** Marked done`).
   - If the user asks for something else (button or typed), do it, update the card again, and repeat this check-in. Keep going until they choose "That's all, keep it in <list>", "Mark card as done" or the snooze until the meeting day.
   - Skip and Snooze buttons never get a check-in. A suggested-action click gets one only if the card still has parts left (several items in the title, or unchecked checklist items). If the card is fully done and the user wrote nothing themselves, go straight to the recap and the next card.
4. **Update the card** (except when skipped; for a snooze this is the **Snooze fast path**, nothing more): dated comment and metadata (e.g. add a new person we emailed to People). Then set the list:
   - **When we've just sent an email or request in this review** (an email or message was sent, a request or form was submitted, an appointment was requested, a document is with someone else), the card now waits on someone else: snooze it until reply: wake on a reply from the person we contacted, with a follow-up date **the user chooses** (recommend 7 days, and never later than 1 day before the deadline; see **The date is always the user's decision**). `Snoozed from` is the card's current list, and the card moves to the right Snoozed list. Mention it in the recap.
   - **A booked or confirmed meeting is not waiting on anyone:** don't snooze the card for it. If nothing else is open, the check-in offers the snooze until the meeting day (see Step 3.3).
   - **Move it back to `In Progress`** when a reply arrived or something new needs action from the user.
   - Move it to `Done` when the task is finished (check its checklists first, see **Card content check**).
5. **Recap the card before showing the next one.** The recap goes at the top of the next card's SendUserMessage, followed by the next card header. (Exception: if the check-in ran, the recap was already sent with it, so don't repeat it. Just add anything done after that check-in.) Shape:
   ```
   ✅ [card name](link)
   - Sent email to Dana (reply in the existing thread)
   - Logged it in a Trello comment
   - Moved the card to Snoozed/Waiting next week (back in In Progress on Thu Oct 15)
   - Snoozed until Dana replies or Thu Oct 15
   ```
   - One bullet per thing actually done: emails sent, comments added, metadata written, label added, checklist items ticked, card moved (say which list), due date set, meeting and travel events created (with the travel time and where it came from), watcher or reminder created, snoozed until date / reply / meeting day, or "stays in daily review until done".
   - For **Skip for today**, just `⏭️ Skipped [card name]`.
   - Only list things that really happened. If an action failed, say so in the recap.
   - **Don't wait for the writes (Snooze and Skip answers only).** Send the writes and, in the same turn, send the next card's SendUserMessage (its recap on top, then the card header and bullets) and its AskUserQuestion, so the user can read and answer while the writes finish. Before the message after that, check that all the writes succeeded; if one failed, say so at the top of the next message and fix it first. Write recap bullets only for writes you have confirmed; for the ones still running, write what is being done ("Snoozing until ..."). Anything involving an email, a calendar event, a checklist change, a due date or moving a card to `Done` still waits for its result before the next card, as does any answer that needs a date or a check-in from the user.
6. Note what the user accepted, snoozed, skipped, changed or rejected. These become candidate learnings for Step 5.
7. Go to the next card (only once any snooze on this card has a date the user chose, and, if the user wrote anything themselves or the card still has parts left, only after they chose a finishing option in the check-in: "That's all, keep it in <list>", "Mark card as done" or the snooze until the meeting day). Keep prep running ahead (see Step 2b, item 4). If the user asked to stop, don't show the next card.

When all the cards are done, list any remaining unsnoozed cards the user didn't get to and ask (buttons) if they want to go through them now or leave them for tomorrow. (Not after a stop: see below.)

## Stopping the review
The user may not have time to go through everything, so they can stop at any time, during the proposed-task triage or the card-by-card part.

- **How they stop:** by typing something like "stop", "that's enough", "wrap up", "I'm out of time" or "pause", in the "Other" box or as a chat message, on any card or proposal, including right after the brief. Use judgment: if the text is about a card action (e.g. "stop, don't send that email"), it is about that card, not the review.
- **When they stop:**
  1. Stop showing proposals and cards and stop any running prep. Throw the prepped drafts away.
  2. Finish nothing new on the card or proposal in front of them. If an action was only proposed or drafted (an email draft, a meeting, a snooze without a date the user chose), it did NOT happen: say so in one line. Anything already done and confirmed stays as it is.
  3. **Cards and proposals not yet reviewed are left untouched.** No snooze, comment, metadata or list change. Unreviewed proposals stay in `Proposed`. Everything comes back in tomorrow's review.
  4. Don't ask any "anything else?" or "want to continue?" question.
  5. Go straight to **Step 5**, with one extra line: "Reviewed X of N cards and Y of M proposals. Not reviewed: <card links, top few, then 'and K more'>." If they stop before anything was reviewed, just reply with one line ("OK. Say 'let's go through my tasks' anytime to pick this up.") and skip Step 5.
- If the user says they want to continue later in the same chat, pick up with the next unreviewed proposal or card (redo any stale prep first).

## Step 4: Execute what the user chose
- **Emails:** draft them and show the full text (with SendUserMessage) before sending. Never send without the user's yes. Reply in the existing thread. Language: the recipient's language; if the user has a saved learning about language for a recipient or kind of recipient, follow it. **Footer on every email Claude drafts (add it before showing the draft):** on its own line at the very end, after the sign-off, in the email's language: `✨ Drafted with Life Automation, a Claude plugin`. Translate it when the email is not in English. After sending, the card is snoozed until reply, with a date the user chooses (see Step 3.4). A draft made during prep is only a draft: it still gets shown to the user and needs their yes.
- **Trello:** dated comment (`**YYYY-MM-DD:** what happened / next`), update the checklist, move the card, set a due date for deadlines and follow-ups.
- **Meetings:** when a meeting is created on Google Calendar, follow **Meetings and travel**.
- **Waiting on someone:** the snooze until reply covers most cases. Offer a separate watcher (a scheduled Gmail check that logs replies on the card between reviews, paid plans only) only if it's time-sensitive. Create it only after the user agrees.
- **Documents:** find attachments, translate to English (or the user's language), save to Drive or link them on the card.
- **Reminders:** on paid plans, Claude scheduled tasks (phone notification). On non-paid plans, a Trello due date on the card instead. No Google Calendar events for plain reminders unless the user explicitly asks. Real meetings and appointments that need a calendar event are covered by **Meetings and travel**.
- Never post on social media or message anyone except by an email the user approved.

## Step 5: Wrap-up (3–5 lines)
Send with SendUserMessage.
- What changed today (including cards snoozed and skipped).
- Proposals from Gmail: A added, B merged, C dismissed, D already completed, E skipped (only if there were any).
- If the user stopped early: "Reviewed X of N cards and Y of M proposals. Not reviewed: <card links>" (see **Stopping the review**).
- Labels added today (by the Category labels check or at a card), with card links. Only if any.
- Top 1–3 focus items for tomorrow.
- Propose 0–2 new learnings from today (see **Learnings**), including a `Gmail discovery:` learning when the user dismissed several proposals of the same kind. Save only the ones the user confirms.
- Ask about the learnings with AskUserQuestion (one question per learning; skip the question if there are none). Send the wrap-up first, then handle the learnings; that is always the last thing in the run.

## Style
- Concise. Status first, no narration of process.
- Suggestions first, then a question with buttons.
- Link to Trello cards, Gmail threads and Drive files instead of pasting content.
