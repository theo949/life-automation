---
name: "todo-capture"
description: "Log a pasted text message, call note or screenshot onto the right Trello card. Use when a message starts with \"tc:\" followed by what someone sent (SMS, WhatsApp, chat, call notes)."
---

# Capture (tc:)

People reply outside email (text, WhatsApp, calls). This skill puts those replies on the right card on the user's Life Automation board so they are not lost, and wakes the card if it was waiting for that reply.

## Trigger
A message that starts with `tc:`. Everything after it is content for Trello. Words the user types before the pasted text are hints (a name, a card topic), e.g. `tc: Maria, car seat: <pasted text>`. The content can be pasted text or a screenshot. If it is a screenshot, read the text from it; if it cannot be read, say so and ask for the text.

This is an explicit instruction to write to Trello, so no extra confirmation is needed for the comment, the People update or the wake-up described below. Anything else (sending a message, creating a calendar event, changing a due date) still needs the user's yes.

## Steps

1. **Load settings and learnings.** Read `/areas/life-automation.md` (if missing, say "Type /todo setup first" and stop), then `/areas/life-automation-learnings.md` and follow it (for example rules about sensitive cards).

2. **Find the card.** Work out who the text is from and what it is about, using the hints and the content.
   - Find the Google Sheet titled "Life Automation Log" (by title) and check its **Card index** tab: match the person against `people_emails` and the topic against `title`. If the sheet or tab is missing or empty, skip this.
   - Also use trelloSearch on the person's name and the topic. Some sensitive cards can be missing from the Card index, so a Trello search is the fallback.
   - Look only at open cards: ignore `Done` and `Proposed`. Cards in the Snoozed lists are valid targets.
   - Read the best candidates fully (description, People, latest comments) before deciding.

3. **Decide.**
   - **One clear match** (the person is on exactly one open card and the topic fits, or the user named the card): use it.
   - **Several plausible cards or an unclear topic:** ask with AskUserQuestion, up to 3 candidate cards plus "New card".
   - **No match:** ask with AskUserQuestion which list the new card goes to (`TO DO` recommended, then `In Progress`) with a short suggested title. Create it only after the answer. Do not put it in `Proposed`: that list belongs to the discovery helper.

4. **Log it on the card.**
   - Add a Trello comment: `**YYYY-MM-DD:** Text from <Name> (<channel if known, e.g. SMS, WhatsApp>): "<the text, verbatim>"`. For a screenshot, write the transcribed text and add "(from screenshot)". Never paraphrase or shorten the person's words.
   - If the sender is not in the card's metadata block `People`, add them (name, phone number or handle if shown, role). Follow the metadata block rules of the `todo-review` skill: edit only text inside the `--- 🤖 Claude metadata ---` block, create it at the bottom of the description if the card has none, and update `Last updated`.
   - For a new card: title from the content (short, imperative, neutral), the text in a comment, and a metadata block with People.

5. **Wake the card if this is a reply.**
   - If the card is in a Snoozed list and the text is from the person in `Wake on reply from` (or clearly answers what the card was waiting for), remove the snooze lines from the metadata block, move the card to `In Progress`, and add the usual comment about the wake-up.
   - If the text is not an answer to what the card was waiting for, leave the list unchanged and say so.

6. **Do not** send any message or email, create calendar events or reminders, change the title, due date, labels, members or checklists of an existing card, or touch anything other than the comment, the metadata block and the list move above.

7. **Reply in 2-3 lines.** `✅ [card name](link): comment added` plus what else changed (person added, card woken and moved to In Progress). If the text contains something that looks like a task, date or request, add one line suggesting what to do about it and ask. For cards the learnings mark as sensitive, keep the reply short and do not repeat the contents.

## Style
Concise, no narration of the steps. Use buttons for any choice. Link to the Trello card instead of pasting its contents.
