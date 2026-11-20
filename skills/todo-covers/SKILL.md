---
name: "todo-covers"
description: "Life Automation helper: after a review, makes sure every labeled card in TO DO / In Progress (and cards touched in the review) has the right label-based Trello cover, and sets it if not."
---

# Cover sweep (Life Automation helper)

Runs once at the very end of an interactive review (`todo-review`) or a single-card review (`todo-card`), after the Write queue flush report is in. It checks that each labeled card has the right cover and sets it when it does not. It is silent unless it has something to add or something failed. Unattended runs never run it and it changes nothing there.

In this skill "the user" is the person running the review. Write to them as "you". Times are 12-hour with AM/PM.

## Which cards
- **Review:** every card in `TO DO` and `In Progress` (from the list reads the review already holds, in Step 0 order), plus every card the review touched that now sits in a Snoozed list. Never cards in `Done`, `Proposed` or archived.
- **Single-card review:** only that card (not if it went to `Done`).
- Labels come from the list reads or the card read (name and color). Do not read the board again. A card with no label (after the skipped labels below) gets no cover and is left alone.
- Cap: the first 40 cards in that order. Say `and K more next time` in the report if there are more.

## Skipped labels
The settings file `/areas/life-automation.md` may hold a line `Cover skip labels: <name>; <name>`. Default when the line is missing: no label is skipped. A label on that list is ignored for covers: it never makes a cover and never counts toward the first two. A card whose only label is a skipped one gets no cover. A card with a skipped label and another label gets the cover for the other label alone.

## What the cover looks like
- **Key:** the label name in lower case, every run of characters that is not a letter or digit replaced by one `_` (`Learning and growth` becomes `learning_and_growth`).
- **Cards with 1 label:** solid color with the white icon centered. **2 labels:** split 50/50 in the order the labels sit on the card, each half with its own white icon centered in it. **3 or more:** the first two. The file name is the keys joined by `-`, for example `finance-health.png`. Size 1200 x 480 PNG, icon 200 x 200, white.
- **Existing categories keep their exact look:** finance `#1F845A` icon `payments`; health `#C9372C` icon `favorite`; family `#6E5DC6` icon `family_restroom`.
- **Any other label:** the color is the label's own Trello color (strip `_light` and `_dark` from the color name and use the bold variant): green `#1F845A`, yellow `#946F00`, orange `#C25100`, red `#C9372C`, purple `#6E5DC6`, blue `#0C66E4`, sky `#227D9B`, lime `#4C6B1F`, pink `#AE4787`, black `#626F86`. A label with no color uses `#626F86`.
- **Icon for a new label:** first read the settings line `Cover icons: <key>=<icon>; ...`. If the key is not there, pick the best-fitting Material Design filled icon from the label name (for example `job_search` gets `work`), check that the file exists in the package (see Building), and save it: memory_str_replace the `Cover icons:` line in `/areas/life-automation.md` (memory_append the line if it is missing). Never ask the user. Mention the choice once in the report (`JOB SEARCH: orange, work icon`).

## Check (read-only, one parallel batch)
For every card in scope work out the expected file name, then read its attachments (trelloReadCard `list_attachments`, all cards in one parallel batch). An attachment with exactly the expected name means the cover is right: leave it. Anything else means the cover is missing or outdated: it needs the new one. Do not delete old attachments. If nothing needs a cover, stop silently.

## Building the images (in the workspace)
Build each distinct file name once, even if many cards share it.

```bash
mkdir -p /tmp/covers && cd /tmp/covers
npm pack @material-design-icons/svg@0.14.13 >/dev/null && tar xzf material-design-icons-svg-*.tgz
pip install cairosvg pillow --break-system-packages -q
```

```python
import glob, io, base64, json, cairosvg
from PIL import Image
W, H, IS = 1200, 480, 200
def icon(name):
    p = [f for f in glob.glob(f'package/**/{name}.svg', recursive=True) if '/filled/' in f][0]
    svg = open(p).read().replace('<path ', '<path fill="#fff" ')
    return Image.open(io.BytesIO(cairosvg.svg2png(bytestring=svg.encode(), output_width=IS, output_height=IS))).convert('RGBA')
def make(parts, out):  # parts = [(hex, icon_name), ...] in label order
    im = Image.new('RGB', (W, H)); seg = W // len(parts)
    for i, (hx, ic) in enumerate(parts):
        w = seg if i < len(parts) - 1 else W - seg * i
        im.paste(Image.new('RGB', (w, H), hx), (seg * i, 0))
        g = icon(ic); im.paste(g, (seg * i + (w - IS) // 2, (H - IS) // 2), g)
    im.save(out)
```

Then base64 each PNG into a JSON map `{file_name: base64}`. If an icon file does not exist, pick another icon name and save that one. If the build fails, report it (see Report) and stop.

## Uploading (Claude in Chrome)
Send one line first with SendUserMessage: `Dressing up N cards with covers (opening a Trello tab in Chrome)...`. Then follow the `chrome-browser` skill: load the Chrome tools in ONE ToolSearch call, `tabs_context_mcp` first, open a NEW tab at `https://trello.com`, run the script below with `javascript_tool` in that tab (fill `imgs` and `jobs`), and close the tab when done. If Chrome is not available, use the built-in browser if it is signed in to Trello. If neither works, go to Report with the failure.

```js
(async () => {
  const imgs = { /* 'finance-health.png': '<base64>' */ };
  const jobs = [ /* {id: '<card object id>', name: 'finance-health.png'} */ ];
  const dsc = (document.cookie.split('; ').find(c => c.startsWith('dsc=')) || '').slice(4);
  const out = [];
  for (const j of jobs) {
    const bin = Uint8Array.from(atob(imgs[j.name]), c => c.charCodeAt(0));
    const fd = new FormData();
    fd.append('file', new File([bin], j.name, { type: 'image/png' }));
    fd.append('name', j.name);
    fd.append('setCover', 'true');
    fd.append('dsc', dsc);
    const r = await fetch('https://trello.com/1/cards/' + j.id + '/attachments', { method: 'POST', body: fd, credentials: 'include' });
    out.push(j.id + ':' + r.status);
  }
  return out.join(',');
})()
```

Status 200 means the cover was set. Any other status is a failure for that card. Do not retry in a loop; retry a failed card once.

## Report
Send ONE message with SendUserMessage, only when something was added or failed:
- `🎨 Added covers to N cards: [card](link), [card](link).` (add `JOB SEARCH: orange, work icon` for a new label)
- Failures: `Couldn't dress up: [card](link) (<short reason>). Say "retry" and I'll give it another go.` or, when no browser could be used, `Cover sweep skipped: no browser is signed in to Trello. N cards still need a cover: [card](link), ...`.
- Never mention a card that was already right.

## Rules
- Never change anything on a card except adding the cover attachment.
- Never touch cards outside the scope above, and never delete attachments.
- No Trello description, comment, label or list change comes from this skill.