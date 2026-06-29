# Gmail-formatted clipboard prompt

A reusable prompt for converting markdown / chat email-body content
into properly formatted HTML on the macOS clipboard, so that pasting
into Gmail's compose window renders real tables, bullets, bold, etc.
— not raw markdown syntax.

Not a skill yet. Keep here until it earns enough reuse to graduate.

---

## The prompt (copy-paste into a new chat)

Paste this verbatim, then add your email body underneath:

```
Take the email body below and copy it to my macOS clipboard as HTML
so that pasting into Gmail's compose window renders real tables and
formatted text (not raw markdown).

Rules:
- Convert markdown tables to <table> with thin gray borders (1px
  solid #999), 6px cell padding, a light gray header row
  (background #f0f0f0), and a sans-serif font (Arial, Helvetica,
  sans-serif, 14px).
- Convert markdown lists to <ul>/<li>.
- Convert markdown **bold** and *italic* to <b> and <i>.
- Cap body width at 640px via style="max-width:640px;..." on the
  outer <body> so the layout works on desktop and mobile.
- Keep real UTF-8 punctuation for em dashes (—), en dashes (–),
  and curly quotes where natural. Do not entity-escape them.
- Inline all CSS via style="..." on each element. Do not emit
  <style> blocks or class attributes — Gmail strips them.
- A minimal <html><body>...</body></html> wrapper is fine.
- Write the HTML to /tmp/email-body.html, then put it on the
  clipboard with the macOS HTML clipboard type using osascript:

    osascript <<'OSA'
    set htmlData to (read (POSIX file "/tmp/email-body.html") as «class HTML»)
    set the clipboard to htmlData
    OSA

- Confirm with a one-line message naming the byte count copied. Do
  not paste back a preview of the HTML.

Email body to convert:

[paste email content here]
```

---

## Reference (for me, not the prompt)

### Why the HTML clipboard type, not pbcopy or RTF

- `pbcopy` with plain markdown puts raw `|...|` syntax on the
  clipboard. Gmail shows literal pipes; tables don't render.
- `pbcopy` with RTF (via `textutil -convert rtf | pbcopy`) works in
  TextEdit and some clients, but Gmail's compose window strips much
  of the styling, especially table borders.
- `osascript` with `«class HTML»` puts HTML-typed data on the macOS
  pasteboard. Gmail's rich-text compose recognizes this type and
  pastes rendered HTML, including bordered tables.

### Common pitfalls

- **Em dashes (—) inside shell heredocs.** Use a single-quoted
  heredoc (`<<'EOF'`) so the shell doesn't mangle UTF-8.
- **Inline styles, not classes.** Gmail strips `<style>` blocks and
  class selectors. Apply CSS as `style="..."` on each element.
- **Border collapse must be inline.** Set
  `style="border-collapse:collapse;"` on the `<table>` and
  `style="border:1px solid #999;"` on every `<th>` and `<td>`.
  Class-based borders disappear.
- **No `<script>`, no `<style>`, no `<link>`.** Gmail removes them.
- **Width control.** Use `max-width:640px` on `<body>` or a wrapping
  `<div>` so the layout doesn't sprawl on wide desktop windows. The
  640px figure matches typical Gmail rich-text body widths and
  reflows cleanly on mobile.

### RTF fallback (if Gmail isn't the target)

Some clients accept RTF but not HTML from the clipboard. For those:

```bash
textutil -convert rtf -stdin -stdout < /tmp/email-body.html | pbcopy
```

Gmail accepts HTML better than RTF, so HTML is the default for Gmail
specifically. Use RTF when the target is Apple Mail's plain compose,
older mail clients, or word processors where HTML paste behaves
inconsistently.

### Where else this technique works

- **Apple Mail** compose window (rich text) — HTML pastes cleanly.
- **Notion** — HTML paste preserves tables, lists, and formatting.
- **Slack** canvas / posts — most formatting carries.
- **Google Docs** — HTML pastes as styled document.
- **Confluence**, **Linear** comments, **Obsidian** — mileage varies;
  test once per client.

### What to leave to the receiving AI

The prompt deliberately tells the AI to confirm with a byte count
and skip pasting a preview. Long HTML previews in the chat are noise
once the clipboard is set. If a preview is wanted for debugging,
the AI can be asked separately ("show me the HTML you generated").
