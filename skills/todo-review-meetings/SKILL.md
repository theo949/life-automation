---
name: "todo-review-meetings"
description: "Helper skill loaded by todo-review-actions, todo-review-scan and todo-card when a meeting is created, moved, removed or found stale (travel events, meeting cleanup, the Events list, Step 2a). Not for direct use."
---

# Daily Task Review: meetings (helper)

This helper is loaded with the Skill tool when a meeting is involved, once per run. It is not started by the user. Everything in the core skill (`todo-review`) and in `todo-review-actions` still applies (typed menus, `Queued:` recaps, the Write queue, 12-hour times, no narration).

## Meetings and travel
This applies **every time Claude creates a meeting (a Google Calendar event) as part of the review**, including errands, appointments, calls moved to a new slot, and anything else that lands on the calendar. It applies in every flow that uses this skill and the core, including the single-card review. If Google Calendar is not connected, say so once, offer to note the meeting on the card instead, and skip everything below.

1. **Decide with AI whether the meeting is in person.** Read the title, description, location, any email thread and the card, and use judgment. In person: a physical address or venue, a clinic, office, pharmacy, notary, café or restaurant, "come to", "visit", an appointment at a place. Not in person: a video link (Meet, Zoom, Teams), a phone call, "online", "remote", or no sign of a physical place. If it's truly unclear, treat it as not in person and don't ask, unless the event is important enough that a missing travel block would be a real problem; then ask a typed yes/no question (see **Choices**).
2. **If it is NOT in person,** create only the meeting. No travel event.
3. **No home address saved:** create only the meeting, never a travel event. Mention once per review, in the recap, "No travel event, because no home address is saved (add one with /todo settings)." Items 4 to 8 below are skipped.
4. **If it IS in person and a home address is saved, also create a travel event** on Google Calendar:
   - **From address (always):** the saved home address.
   - **To address:** the meeting's address (the event location, or the address found in the email or card). Use the full street address when you have it.
   - **Travel time:** calculate it for **public transport** (bus, tram, metro, walking included) from the from address to the meeting address, for a trip that **arrives by the meeting's start time**. Use a browser (Google Maps directions in transit mode with "arrive by" set to the meeting time, or another reliable public transport route planner) and read the duration. Use the actual door-to-door time the planner shows. Don't add extra buffer on your own.
   - **If the meeting has no address,** or the route lookup fails, or no browser is available, default to **30 minutes** of travel time. A missing browser is never a reason to skip the travel event.
   - **Travel event:** title `Travel: <meeting title>`, ends exactly when the meeting starts, starts travel-time minutes before. Location = the meeting address (or blank if none). Description: from address, to address, the travel time and whether it came from the route planner or the 30-minute default, plus a one-line route summary (e.g. "Bus 12 → Metro line 1") when you have it. Same calendar as the meeting. No guests, even if the meeting has guests.
   - If the travel window overlaps another event on the calendar, still create it and flag the overlap in the recap.
5. **Recap:** in the card recap (core Step 3.5), list both, e.g. "Added meeting Thu Oct 15, 2:00 PM (in person, Pharmacy X)" and "Added travel 1:25–2:00 PM (35 min by public transport)". For a default, say "30 min default, no address".
6. **Metadata:** add the meeting to the card's `Events:` list as usual, with the calendar event link at the end of its line. The travel event is not listed there, but if the meeting is later moved or deleted, update or delete its travel event too (and ask before deleting, as with stale meetings).
7. If the meeting is moved to a new time, recalculate the travel time for the new time and move the travel event with it.
8. **Check existing meetings for a missing travel event (card walkthrough in Full mode only, never in unattended runs; Quick mode does not look).** While `todo-review-scan` re-checks a card's listed meetings against Google Calendar, look at each meeting that is in the future, within the next 14 days, and in person (same judgment as item 1). Use the calendar data you already fetched; do not scan the whole calendar, only the meetings listed on cards. A travel event is already there if an event whose title contains "travel" (e.g. `Travel: <title>` or just `travel`) ends at the meeting's start time or up to 15 minutes before it. If there is none, the card has an open item: show it as one bullet in the card summary ("No travel event for <meeting>, <date, place>") and put creating it (item 4 rules) in slot 1 of the menu (or in a bullet when slot 1 is something else). After it's created, carry on with the card's normal next step. If the user says they don't want a travel event for that meeting, add a Trello comment `**YYYY-MM-DD:** No travel event wanted for <meeting title> (<date>)` and don't ask again for that meeting. Never create it without the user's yes. This check is part of the Full-mode card prep, so it runs in the background while the user reads the brief.

Outside item 8, this applies only to meetings Claude creates. Don't add travel events for meetings that already exist on the calendar unless the user asks.

## Meeting cleanup
Meeting cleanup asks about **future meetings only**. Never flag, list or ask about removing a meeting that is in the past: old past lines are dropped by the automatic rule in **Keeping the Events list short**, which needs no question. A future meeting is **stale** when ANY of these is true:
- It no longer exists in Google Calendar. Before calling it stale, search the calendar for a moved or rescheduled version; if one exists, propose updating the entry instead of removing it.
- The card is snoozed until a date after the meeting, so the meeting is no longer what the card is waiting on.
- The user said it's done or irrelevant.

Rules:
- **Never remove a meeting without the user's yes.** Collect stale future meetings quietly in the Full scan (`todo-review-scan`), across ALL cards including those in the Snoozed lists, then ask once in **Step 2a**. Quick mode does not look for them; a meeting the user reports as gone is handled the same way at its card.
- **Once confirmed,** remove only those lines from the block (nothing outside the block), update `Last updated`, and add one Trello comment per card: `**YYYY-MM-DD:** Removed meetings: <title> (<date>), ...`.
- **Unattended runs** (scheduled, nobody to answer): change nothing. List the stale future meetings in the brief so the user sees them next time.
- A stale meeting is never shown in a card summary as upcoming or pending, and never counts as news.
- If a stale meeting has a linked travel event (see **Meetings and travel**), include it in the same confirmation and remove it together with the meeting.

## Keeping the Events list short
The `Events:` list is a working list, not a history: the dated Trello comments are the history. Every write of a description sends the whole text, so a long list slows every change on the card.
- **One line per calendar event.** When a meeting moves to a new date or time (the same calendar event), change its existing line to the new date and time, keep the calendar link, put it back in date order, and do not leave the old date behind as a separate line. The dated Trello comment about the move records the old date. Only a different calendar event gets a line of its own.
- **A meeting the user cancels or asks to remove** (including when Claude deletes its calendar event): remove its line from `Events:` in the same write, and name it in the card's dated Trello comment (`Removed the <date> <title> meeting`). Never leave a `(cancelled ...)` line behind.
- **Old past lines (automatic, no question).** A past meeting's line is dropped once it has a recorded outcome (`(done)`, `(not done)` or `(didn't happen)`) and its date is more than 14 days ago. The Trello comments written when the outcome was recorded keep the history. Do this only while the block is being written anyway for another reason: dropping old lines alone never causes a write and never changes `Last updated`. A past line with no recorded outcome stays. A line for a meeting in the last 14 days is never dropped.

## Step 2a: Confirm meeting cleanup
Only in Full mode, and only if the scan found stale future meetings (see **Meeting cleanup**) and the user is present. The card prep is already running (core Step 2b). Write ONE closing reply, typed (see **Choices**): the list of each affected card with its stale future meetings and why (deleted from calendar, before snooze date, or the user called it irrelevant), a blank line, the question line `Remove N stale meetings?`, a blank line, the menu line and the prompt line (`Type 1–2, or just tell me what you have in mind.`):

`① Yes, remove them · ② Keep them for now`

- **1 Yes, remove them** removes the lines from card metadata and logs a comment on each card (say this in the list above the question, not in the menu).
- **2 Keep them for now** changes nothing; they are asked about again next review.
- Anything else typed (e.g. "keep the Dec 3 one") is followed exactly.

Queue the removals as described under **Meeting cleanup** (a Google Calendar deletion is done at once), then continue to Step 2c (or core Step 3 if there are no proposals), with no extra question. Include them in the Step 5 wrap-up.