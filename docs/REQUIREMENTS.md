# Requirements

Personal, sideload-only Android enforcement app. Stock Android leaves too many escape hatches; this product treats turning the watcher off like unlocking a site.

## Goals

- Cut doomscrolling without a total ban.
- Per-site **use slices**, **cooldowns**, and a **daily cycle cap**.
- Friend-held override key the user never sees (server-sent SMS, not from this phone).
- Device Owner hard mode so uninstall / clear-data are blocked.

Example: `facebook.com` — 20 min on / 2 h off / max 3 cycles per day → at most 60 minutes of use spread over ~7 hours, then blocked until reset.

## Non-goals

- Play Store listing or Play policy packaging
- Accounts or cloud sync of usage data
- Network-wide DNS/VPN as the primary mechanism
- Root
- Stopping factory reset (accepted escape)
- Perfect idle detection on every OEM

## Functional requirements

### Sites and surfaces

- User defines watched domains (normalized registrable domain).
- Subdomains match by default (`m.facebook.com`, `www.facebook.com`).
- A watched domain is enforced in:
  - all installed browsers, including Chrome
  - official apps mapped to that domain (Facebook app ↔ `facebook.com`, etc.)
  - unknown browsers: if the URL cannot be read, fail closed while any site is in cooldown or blocked-until-reset (default on)
- Unwatched sites stay unrestricted.

Early draft said "non-Chrome browsers only." Anti-bypass requires Chrome and mapped apps to be in scope for **enforcement** on watched domains. Tracking other Chrome sites is not required.

### Time model

- Limits are per site, not shared across sites.
- Each site has: slice length, cooldown length, max cycles per day.
- Active time = watched surface is foreground AND screen on AND (optional) last input within idle pause (default 60s).
- Leaving mid-slice **pauses** the slice; returning continues the same slice until it is exhausted.
- Exhausting a slice increments the cycle counter and starts cooldown.
- After the last cycle, state is `blocked_until_reset` until local reset hour (default midnight).
- Unused cycles do not roll over.

### Enforcement when limited

- Redirect / leave the site: browser tab to home or blank (`about:home`, `about:blank`, `chrome://newtab` as available).
- Mapped apps: blocking overlay until the user leaves the app.
- Overlay is not dismissible by Back. No local snooze / ignore-today / visible PIN.
- Overlay shows site, phase, remaining time, and **Request unlock** (notifies the friend only).

### Trusted friend / key

- Enroll a trusted phone number.
- Backend generates a high-entropy key and SMS it from a service number (Twilio or similar) to the friend.
- The key is never rendered in the app, notifications, or the user's SMS Sent folder.
- Changing the trusted number requires the current key.
- Request unlock does not grant time. Friend approves via service SMS or by dictating the key once into a one-shot field that is not persisted.
- Grant types (v1): extra cycle; disable that site until reset; pause N hours (1 or 2).
- Overrides are logged and cannot be deleted without the key.

### Anti-bypass

- Hard mode requires Device Owner (`adb shell dpm set-device-owner …`).
- Device Owner: forbid uninstall and clear-data of this package.
- Soft mode (no Device Owner) runs with a persistent warning that Settings can still kill the watcher.
- Accessibility drop → fail closed on watched sites until the service returns.
- Policy edits while enforcement is on require the friend key.
- Factory reset, Safe Mode, and friend collusion are accepted holes.

## Permissions

- Usage access
- Accessibility
- Display over other apps
- Foreground service
- Battery optimization exemption
- Device Owner (hard mode)
- Network for enroll / notify-friend / confirm only

Setup is incomplete until those plus trusted-number enrollment are done.
