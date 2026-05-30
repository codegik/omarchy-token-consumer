<p align="center">
  <img src="icon.png" alt="omarchy-token-consumer" width="128">
</p>

<h1 align="center">omarchy-token-consumer</h1>

<p align="center">
  A <a href="https://github.com/Alexays/Waybar">Waybar</a> module for
  <a href="https://omarchy.org/">Omarchy</a> that shows your live
  <strong>Claude Code rate-limit usage</strong> right in your status bar.
</p>

---

## What it's for

If you use Claude Code, you have rate limits — and it's easy to burn through
them without noticing. This module puts a small indicator in your Waybar so you
always know where you stand:

```
󰚩 5%
```

Hover it for the full breakdown — **session (5h)**, **weekly**, **Opus**, and
**Sonnet** — each with its reset time. The number turns **yellow at 80%** and
**red at 100%**, so a glance is enough.

Under the hood it calls the exact same endpoint Claude Code's own `/usage`
command uses (`GET https://api.anthropic.com/api/oauth/usage`) and reuses the
OAuth token already on disk at `~/.claude/.credentials.json`. That means **no
API keys, no config files, and no log scraping** — it's a single dependency-free
Python script (stdlib only).

## Demo

<video src="https://github.com/codegik/omarchy-token-consumer/raw/main/recording-20260529-233747.mp4" controls width="640"></video>

## Install

It's a single Python script — drop it on your `PATH`:

```bash
git clone https://github.com/codegik/omarchy-token-consumer.git
cd omarchy-token-consumer
install -m 0755 omarchy-token-consumer ~/.local/bin/
```

Then add the module to Waybar. In `~/.config/waybar/config.jsonc`, add
`"custom/tokens"` to one of your `modules-*` arrays, and define it:

```jsonc
"custom/tokens": {
  "exec": "$HOME/.local/bin/omarchy-token-consumer",
  "return-type": "json",
  "interval": 300,
  "signal": 11,
  "format": "{}",
  "on-click": "xdg-open https://claude.ai/settings/usage"
}
```

Optionally, add color coding to `~/.config/waybar/style.css`:

```css
#custom-tokens { margin: 0 7.5px; }
#custom-tokens.warning  { color: #d4a72c; }  /* >= 80% on any limit */
#custom-tokens.critical { color: #a55555; }  /* >= 100%, or no auth  */
```

Apply it:

```bash
omarchy restart waybar
```

Force an immediate refresh anytime with `pkill -RTMIN+11 waybar`.
