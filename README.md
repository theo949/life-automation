# Life Automation

Life Automation turns your inbox into a Trello to-do board and walks you through a daily review with Claude. You put tasks on Trello, Claude finds more in your email, and each morning you go through them together. Claude can set up meetings, draft emails for your approval and do research for you.

## What you need

- A Trello account (the free plan works)
- These connectors in Claude: **Trello**, **Gmail**, **Google Drive and Sheets**
- Optional: **Google Calendar**, so tasks can become calendar events
- Optional: a Claude plan with scheduled tasks, so the daily prep runs before you wake up. Without it, you run `/todo` by hand and each run starts with a quick scan of your email.

To add a connector, open **Settings → Customize → Connectors**, find it in the list and press **Connect**.

## Install

**In the Claude app:** Settings → Customize → Plugins → Add → Add from a repository, then enter `theo949/life-automation` and install Life Automation.

**In Claude Code:**

```
/plugin install life-automation --marketplace theo949/life-automation
```

## Get started

1. Install the plugin, then type `/todo` in Claude. Setup runs once: it checks your connectors, asks a few questions, then builds your Trello board, a Google Sheet log and (on paid plans) the daily schedule.
2. Open your new Trello board. You'll see a list called **Proposed** with tasks found in your email. They are only suggestions. Nothing happens until you approve them in your first `/todo` review.
3. Type `/todo` in Claude and go through your tasks.

**Tip:** the Claude browser extension makes this smoother. With your Trello board open, press Cmd+E (Mac) or Ctrl+E (Windows), or click the Claude icon at the top right of your browser, to open Claude beside your board. Without the extension, keep a browser window with Trello next to the Claude app.

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

Your home address, if you give one for travel times, is kept in your Claude memory, never in Trello or the sheet.

## Usage

Like any skill, Life Automation uses your Claude plan's usage, so check it now and then. The first email scan is the biggest.

## Skills in this plugin

`todo` (entry point), `todo-setup`, `todo-review`, `todo-review-card`, `todo-capture`, and the helpers `todo-prioritizer`, `todo-discovery` and `todo-prep`.

## Videos

Short walkthrough videos: coming soon.

## License

MIT
