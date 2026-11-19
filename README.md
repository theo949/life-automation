# Life Automation

Life Automation turns your inbox into a Trello to-do board and walks you through a daily review with Claude. You put tasks on Trello, Claude finds more in your email, and each morning you go through them together. Claude can set up meetings, draft emails for your approval and do research for you.

## What you need

- A Trello account (the free plan works)
- Claude's memory turned on (Settings → Your Claude → Memory → "Generate memory from chats"). See [Claude memory](#claude-memory) below.
- These connectors in Claude: **Trello**, **Gmail**, **Google Drive**, **Google Sheets**
- Optional: **Google Calendar**, so tasks can become calendar events
- Optional: a Claude plan with scheduled tasks, so the daily prep runs before you wake up. Without it, you run `/todo` by hand and each run starts with a quick scan of your email.

To add a connector, open **Settings → Customize → Connectors**, find it in the list and press **Connect**.

### Claude memory

Life Automation needs Claude's memory turned on. It keeps two things there.

- **Your settings:** your review time and days, time zone, email lookback, home address (if you give one), whether you're on a plan with scheduled tasks, and which setup step you reached, so an interrupted setup can continue.
- **What it learns about you:** senders to ignore, people who matter most, the language to use with a recipient, and how you like cards handled. This is what makes the reviews better over time.

The scheduled helpers on paid plans run on their own, in a new session each time, and read the same saved settings. Without memory they can't know your review time or lookback.

To turn it on, open **Settings → Your Claude → Memory** and switch on **Generate memory from chats**. Then type `/todo`, or `/todo setup` if setup already started.

That switch lets Claude generate memory from all your chats, not only Life Automation. To limit it, open **Your memory → Manage** to see, edit or delete anything Claude has saved. Life Automation only needs its own entries, named "Life Automation settings" and "Life Automation learnings". Your home address is saved there, never in Trello or your sheet.

### Permissions to allow

Each connector asks you to approve what Claude may do. Here is what to tick and why.

- **Trello:** pick your workspace, and leave both **Read** ("Let AI agents view your existing work and use it as context, without creating or changing anything") and **Write** ("Let AI agents create and update work on your behalf, using the same permissions you already have") checked. Setup needs both to build your board. If the workspace list says "No workspace available", create a free workspace at trello.com first. Being a guest on someone else's board isn't enough.
- **Gmail:** tick the first two boxes.
  - "View your email messages and settings." Used to find to-dos in your email.
  - "Manage drafts and send emails." Used to send the emails you approve.

  The third box, "Read, compose and send emails from your Gmail account", isn't needed. It appears because the Gmail connector is general-purpose, with actions like labeling and trashing that Life Automation never uses. Life Automation never labels, archives or deletes email, and never sends without showing you the exact text first.
- **Google Drive:** tick both boxes.
  - "See and download all your Google Drive files." Read-only. Used to open Google Docs, Sheets and files linked on your Trello cards, so the review can tell you what changed in them and prepare the card ahead of time. It reads these files and never changes them.
  - "See, edit, create and delete only the specific Google Drive files that you use with this app." Limited to files Claude itself creates or opens. Used to create your Life Automation Log sheet and keep it up to date, and to create the Google Docs that longer `#offline` research results are saved in. It doesn't touch your other files, and Life Automation never deletes anything.

  The second box alone is enough for setup and the daily run, and is the more private choice. Without the first, Claude can't read Google files linked on your cards, so the review skips them and you open those files yourself. For the full experience, tick both.
- **Google Sheets:** a separate connector from Google Drive. Google shows one screen, titled "Claude for Google Drive wants additional access to your Google Account" (the Sheets connector uses the same Google app as Drive), with one permission: "See, edit, create and delete all your Google Sheets spreadsheets." Allow it. Life Automation uses it to read and write your Life Automation Log sheet, which keeps its records. Without it setup stops at the log sheet, because the scan has nowhere to save what it found. Google words this permission broadly (all your spreadsheets), and Life Automation only reads and writes its own log sheet.

## Install

**In the Claude app:** Settings → Customize → Plugins → Add → Add marketplace, then enter `theo949/life-automation` as the marketplace. Then open Discover, find Life Automation and install it.

**In Claude Code:**

```
/plugin install life-automation --marketplace theo949/life-automation
```

## Get started

1. Install the plugin, then type `/todo` in Claude. Setup runs once: it checks your connectors, asks a few questions, then builds your Trello board, a Google Sheet log and (on paid plans) the daily schedule.
2. Open your new Trello board. You'll see a list called **Proposed** with tasks found in your email. They are only suggestions. Nothing happens until you approve them in your first `/todo` review.
3. Type `/todo` in Claude and go through your tasks.

**Tip:** the Claude browser extension makes this smoother. With your Trello board open, press Cmd+E (Mac) or Ctrl+E (Windows), or click the Claude icon at the top right of your browser, to open Claude beside your board. Without the extension, keep a browser window with Trello next to the Claude app.

## How the daily run works

On a plan with scheduled tasks, three helpers run in the hour before the review time you choose, so your review is ready when you sit down:

1. **Prioritizer:** ranks your cards and wakes snoozed ones.
2. **Discovery:** scans your recent email for new to-dos and adds them to your Proposed list.
3. **Prep:** reads ahead on each card so the walkthrough is fast.

At your review time, Claude shows a short brief. Type `/todo` to go through your tasks, then or any time you like. Without scheduled tasks there are no helpers, so each `/todo` run does this prep first and takes a minute or two longer.

## If it feels slow

- Look back a shorter time in your email. The default on a free plan is 3 days.
- Review a few cards at a time and stop when you like. Your place is saved.
- A paid plan with scheduled tasks does the prep before you sit down, so the review opens quickly.

## What Claude writes on your cards

Open a card in Trello after a review and you'll see two kinds of notes from Claude. They make each review faster and show you what happened and why.

**The metadata block.** At the bottom of a card's description, Claude keeps a block of what it has learned about the card: the people involved, linked documents, calendar events, related emails and other links. Your own notes stay above it. Claude only changes what is inside the block and never the text above it. For example:

```
--- 🤖 Claude metadata ---
Last updated: 2026-11-18
People:
- Dr. Petrova <clinic@example.com>, dentist
Events:
- 2026-11-25 11:00: Dentist follow-up | https://calendar.google.com/...
Emails:
- Dentist confirmed the follow-up (Nov 17): https://mail.google.com/...
--- end ---
```

The sections are People, Documents, Events, Emails and Others, always in that order. A section with nothing in it is left out. While a card is snoozed, the block also holds the snooze date and the list the card returns to. On a paid plan the prep helper fills the block in ahead of time. Without it, the review fills it in as you reach each card. Leave the block to Claude and write your own notes above it, because Claude rewrites the block when it next updates the card.

**Comments.** When something happens to a card, Claude adds a short dated comment. For example, "Snoozed until 2026-11-25", "Meeting happened, work done", or a note on what is still left to do. You get a history of the card in Trello itself.

**What stays under your control.** Titles, due dates, checklists, labels and lists change only when you say yes during a review. Claude shows you what it wants to change and waits for your answer.

## Commands

| Command | What it does |
| --- | --- |
| `/todo` | Starts your daily review (the first run does setup) |
| `/todo card` or "review this card" | Reviews just the card you have open |
| `tc: <text>` | Files a text message, chat reply or call note onto the right Trello card |

## What it reads, creates and never does

- **Reads:** your email from the last week or two (you choose how far back), your Trello board and, optionally, your calendar.
- **Creates:** a Trello board with cards and comments, a Google Sheet log and, on paid plans, a daily schedule.
- **Never:** sends an email without showing you the exact text first.

## Privacy

- Life Automation has no server of its own. Everything runs in your Claude account, through the connectors you approved.
- It reads your recent email, your Trello board and, if you connect it, your calendar.
- Your home address, if you give one for travel times, is saved in your Claude memory, never in Trello or your sheet. You can enter the nearest metro or bus stop instead.
- It never sends an email without showing you the exact text first.

## Usage

On a paid plan, Life Automation uses your Claude plan's usage, so check Settings → Usage now and then. The first email scan is the biggest. On a free plan you can't see your usage. You'll know you've hit the limit when Claude says you're out of messages until a set time. Your place is saved: setup and reviews pick up where they stopped, so wait for the reset and type `/todo`.

## Skills in this plugin

`todo` (entry point), `todo-setup`, `todo-review`, `todo-review-card`, `todo-capture`, and the helpers `todo-prioritizer`, `todo-discovery` and `todo-prep`.

## Videos

Short walkthrough videos: coming soon.

## License

MIT
