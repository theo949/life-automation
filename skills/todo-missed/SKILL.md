---
name: "todo-missed"
description: "Life Automation: the user says discovery missed an email that should have become a Proposed card; logs the miss and learns from it."
---

# Discovery missed this

Turns one missed email into something discovery learns from. In this skill "the user" is the person running Life Automation; write to them as "you". Keep it short: three replies at most.

Never send, draft, label, archive or delete any email. Never store health details, finances or other sensitive personal details: for a health sender write `[health sender]` and no specifics.

## Step 1: Find the email
Read `/areas/life-automation.md` (time zone, Trello board, log sheet) and `/areas/life-automation-learnings.md` in one parallel batch. Then find the thread from what the user gave you: a Gmail link (use its thread ID), or words from the subject or sender (search Gmail, newer_than:30d). If several threads match, show up to 3 as a typed menu and let the user pick. Read the chosen thread in full (PLAIN_TEXT).

## Step 2: Why was it missed
Convert the thread ID to hex if needed. Look it up in this order and stop at the first hit:
1. **Skipped** tab (columns found_date, thread_id, reason_code): the reason code is the answer.
2. **Proposals** tab or **Card index** tab: it was not missed, it is already on a card. Tell the user which card and stop.
3. No row: say it was not logged, and name the likeliest reason from the thread itself: it sits in Promotions, Social or Forums (those are never read), it is older than the 7-day log, or it did not match any of discovery's rules (for example: own message last, FYI with no question, automated).

## Step 3: Ask why it mattered
One reply: show the email in one line (sender, subject, date), the reason it was skipped, then ask `In a few words: why should this have been a card?` and offer `1 Add it to Proposed now · 2 Just log it · 3 Skip`. If the user types words instead of a number, take the words as the answer to the question and treat it as `2`, then ask nothing more about adding it unless they also asked for it. For option `1` create the card as `todo-discovery` STEP 3 does (title rules, proposal block with `Confidence: normal`, metadata block, a row in the **Proposals** tab with category `missed`); never touch other lists.

## Step 4: Save the miss
Read the learnings file, then memory_append one line (`- [stated] Discovery miss: <date>; <sender address, or domain for a business>; arrived as <direct / self-sent / forwarded / other>; skipped as <reason code or likely reason>; user said: "<their words, short>"`). Create the file with memory_write if it does not exist (frontmatter: name `life-automation-learnings`, description "How the user likes the daily Trello task review run: confirmed preferences and learnings", sources [cowork]).

## Step 5: Propose a rule after repeats
Count the `Discovery miss:` lines (including the new one) with the same reason code, or the same sender address, or the same arrival type. When 3 or more share one of these, write the rule in one plain sentence (for example: `Propose emails from <address> even when they have no question`) and ask `Want me to save this rule?` with `1 Save it · 2 Skip`. On yes, memory_append `- [stated] Gmail discovery include: <the rule>`. When fewer than 3 share one, say nothing about rules. Never write a rule the user did not confirm.

## Final reply
One or two lines: what was logged, whether a card was added, and whether a rule was saved.