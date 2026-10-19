---
name: "todo-review-card"
description: "Run the Life Automation review on just the one Trello card the user has open. Use for 'this card only', 'just this card', or 'review this card' when Claude is beside an open card."
---

# Review: Single Card

The `todo-review` skill, narrowed to one card: the one the user has open in Trello. Same rules, same card-level steps, no board walkthrough.

## Start
1. Load the `todo-review` skill with the Skill tool and read it. Everything it says about settings, learnings, the card metadata format, Meetings and travel, Meeting cleanup, Deadlines, Snoozing, Showing text, Buttons and Step 4 applies here unchanged. Where this skill and that one differ, this one wins for scope.
2. Do what `todo-review` says under **Before anything else** (load `/areas/life-automation.md`; stop and point to `/todo setup` if it is missing), then read the learnings file `/areas/life-automation-learnings.md` and follow it.

## Find the card
- If the user names or links a card in chat, use that card.
- Otherwise look at the browser the user is working in (Claude in Chrome tabs context, or the built-in browser). The active tab on a Trello card has a URL like `trello.com/c/<shortLink>/...`. Read that card with the Trello connector (description, checklists, comments, members, due date, current list).
- If no browser is reachable (for example a scheduled run) or the active tab is not a card, ask the user for the card name or link. If they give a name, use trelloSearch. If there are several plausible matches, show the top ones as buttons. Never guess.
- Name the card in the first line of the summary so the user can catch a wrong match.
- If the card is in one of the Snoozed lists (`Snoozed/Waiting this week`, `Snoozed/Waiting next week`, `Snoozed/Waiting later`), say so and ask (buttons) whether to wake it before reviewing.
- If the card has a `Snoozed until` line but sits in a regular list (the user moved it themselves), treat it as woken: clear the snooze lines from the metadata when you next update it, and mention it in the summary.

## Scope: what to skip
- No board-wide reads, no Step 0 ordering, no waking other cards, no Step 2 status brief, no Steps 2a/2b, no "Card X of N", no moving on to a next card.
- The 3-cards-per-day enrichment cap does not apply: the user asked for this card, so enrich it if it has no metadata.
- Meeting cleanup covers only this card's `Events:` list.

## Run
1. **Gather for this card only.**
   - Card has metadata: Step 1a (new emails and calendar events involving the listed people since `Last updated`; re-check listed meetings against Google Calendar, and check each upcoming in-person meeting for a missing travel event, see Meetings and travel item 8).
   - Card has no metadata: Step 1b (30-day window around creation). Write the metadata found right away and say what was added in the summary; don't ask for confirmation first.
   - Card has a linked Google Sheets, Docs or Slides file (either case): run the **Linked Google files** check from `todo-review` (comments read with the comments option on, changed statuses, new since `Last updated`). Show new comments or status changes as the first bullet of the summary, and put what to do about them in the suggested action.
   - Always (either case): run the **Card content check** from `todo-review` (checklists read directly; checklist items, title and description against comments, emails, calendar and activity), and the **Past meetings** follow-up check for this card's listed meetings from the last 14 days whose outcome isn't recorded. Show findings as bullets in the summary and put the update in the suggested action. If the user says something is done, apply **Stated done is the go-ahead** and tick the matching checklist item.
2. **Stale meetings:** if any, send one SendUserMessage and ask one question: "Remove N stale meetings from this card?" (same buttons as Step 2a). Never remove silently.
3. **Summary:** send with SendUserMessage: `**This card:** [name](link) · list · deadline if any`, then 1-3 bullets on where it stands (latest comment, new emails with thread links, what it's waiting on, money or dates, the user's own next step). Only still-future meetings are shown as upcoming. If an upcoming in-person meeting has no travel event, add one bullet for it.
4. **Ask one question** with AskUserQuestion using the Step 3.2 option rules from `todo-review`: the suggested action first (Recommended), then snooze options with real dates in the labels, respecting the deadline rules. Replace "Skip for today" with "Leave as is" (no changes). When the only thing still open on the card is an upcoming meeting, the suggested action is the snooze until the meeting day.
5. **Act** as in Step 3.3, Step 3.4 and Step 4: dated Trello comment, metadata update, list moves (a snooze goes to the right Snoozed list for its date), snooze only with a date the user chose, emails shown in full and sent only after the user's yes. Any meeting created follows Meetings and travel.
6. **Check-in loop:** after any action, or whenever the user typed anything themselves, send a short recap and ask one question that says what was done and where the card now sits: "<what was done>. The card stays in <list>. Anything else on it?" `<list>` is the card's real list after the update; if the card moved (a Snoozed list), say "The card is now in <list>." instead. First work out whether the only thing still open on the card is an upcoming meeting (a future, not stale meeting in `Events:`, no unchecked checklist items, no reply awaited, no email to send, no missing travel event; and the meeting is not today, and its date is not later than 1 day before the card's deadline). Then the options, in this order:
   - **If a meeting is the only thing open:**
     1. **"Snooze until <Day Mon D> (meeting day) (Recommended)"**. Description: "Hidden in Snoozed until the day of the meeting, then back in your daily review." The date is the earliest such meeting's date; the date in the label counts as the user's choice.
     2. **"That's all, keep it in <list>"**. Description: "Card stays open in <list>." Not Recommended in this case.
     3. **"Something else on this card"**. Description: "Keep working on it."
     - **"Mark card as done" is never shown in this case**, because a meeting is still ahead.
     - If the user chooses the snooze: follow **Snoozing** in `todo-review` (metadata, move to the right Snoozed list, comment) and mention it in the recap. Then the card is finished (a Snooze button gets no further check-in).
   - **In every other case:**
     1. **"That's all, keep it in <list> (Recommended)"**. Description: "Card stays open in <list>." Recommended whenever the card is still open work.
     2. **"Mark card as done"**. Description: "Moves it to the Done list (I'll check its checklist first)." Only show it when the work looks finished (nothing left open, no unchecked checklist items, no reply being waited on); when it is clearly finished, make it the Recommended option instead.
     3. **"Something else on this card"**. Description: "Keep working on it."
   - Never label a button plain "Done" or "Finish" unless choosing it moves the card to the Done list.
   - If the user chooses "Mark card as done": run the checklist check first (if items are unchecked, warn and ask before moving it), then move the card to `Done` and add a dated Trello comment (`**YYYY-MM-DD:** Marked done`).
   - Keep going until the user chooses one of the first two options (in the meeting case, the snooze or keeping it in its list). Never end by yourself after something the user typed.
7. **Wrap-up:** a 1-3 line recap of what changed (Step 3.5 shape). Propose 0-2 learnings from this session and save only the ones the user confirms. No "focus for tomorrow" list.

## Unattended runs
If nobody is there to answer, change nothing on the card. Report what you found (new emails, stale meetings, a suggested next step) so the user sees it next time.
