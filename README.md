# omarchy-token-consumer

A [Waybar](https://github.com/Alexays/Waybar) module for [Omarchy](https://omarchy.org/)
showing your **current Claude Code rate-limit usage** as a percentage:

```
󰚩 5%
```

Hover for the full breakdown (session, weekly, Opus, Sonnet) with reset times.

## Demo

<!--
  GitHub only renders a playable video for files served from its attachment CDN.
  To get the URL: edit this README on github.com (or open a new issue/PR comment),
  drag recording-20260529-233747.mp4 into the text box, and GitHub inserts a
  https://github.com/user-attachments/assets/... URL. Paste it below, replacing
  REPLACE_WITH_UPLOADED_URL. The bare URL form on its own line also works.
-->
https://github.com/user-attachments/assets/REPLACE_WITH_UPLOADED_URL

It calls the same endpoint Claude Code's own `/usage` command uses
(`GET https://api.anthropic.com/api/oauth/usage`) and authenticates with the
OAuth token already on disk at `~/.claude/.credentials.json` — no extra setup,
no pricing tables, no log scraping.

## Install

It's a single stdlib Python script — drop it on your `PATH`:

```bash
install -m 0755 omarchy-token-consumer ~/.local/bin/
```

Add `"custom/tokens"` to a `modules-*` array in `~/.config/waybar/config.jsonc`,
then add this module:

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

Append to `~/.config/waybar/style.css` (optional color coding):

```css
#custom-tokens { margin: 0 7.5px; }
#custom-tokens.warning  { color: #d4a72c; }  /* >= 80% on any limit */
#custom-tokens.critical { color: #a55555; }  /* >= 100% on any limit, or no auth */
```

Apply with `omarchy restart waybar`. Force a refresh anytime: `pkill -RTMIN+11 waybar`.

## How it works

On each run the script:

1. Reads the OAuth access token from `~/.claude/.credentials.json`
   (key path: `claudeAiOauth.accessToken`).
2. If `expiresAt` is in the past, displays `?` and asks you to run `claude` to
   refresh — token refresh is delegated to Claude Code itself.
3. Otherwise calls `GET https://api.anthropic.com/api/oauth/usage` with
   `Authorization: Bearer <token>` (5s timeout).
4. The response contains `five_hour`, `seven_day`, `seven_day_opus`, and
   `seven_day_sonnet` entries, each with `utilization` (percent, 0–100) and
   `resets_at` (ISO 8601). The headline number is the highest of the four.

No CLI args, no config file, no network keys to manage.

Inspired by the macOS WidgetKit widget
[`vfurinii/xcode-playground/token.consumer`](https://github.com/vfurinii/xcode-playground/tree/main/token.consumer).
