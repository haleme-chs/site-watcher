# User stories

IDs are stable. Acceptance criteria are the implementation contract.

## Epic 1 — Surfaces

### US-1.1 Enforce watched domains everywhere they appear

As a user, I want a watched site blocked in every obvious place so I cannot switch to Chrome or the official app to keep scrolling.

Acceptance criteria

- Watched domain matches in Firefox, Chrome, Brave, Edge, Samsung Internet, Opera, DuckDuckGo, and other detected browsers.
- Mapped packages (Facebook, Instagram, YouTube, Reddit, …) consume that domain's slice and obey cooldown / blocked-until-reset.
- Unwatched sites in any browser are unrestricted.

### US-1.2 Fail closed on unreadable browsers

As a user, I want unknown browsers not to become a bypass.

Acceptance criteria

- If the foreground app looks like a browser and the URL cannot be read, and any site is in `cooldown` or `blocked_until_reset`, block that app (default).
- Toggle `block_unreadable_browsers` exists; default on.
- Settings list installed browsers and whether URL parsing is supported.

## Epic 2 — User-defined sites

### US-2.1 Add a site

As a user, I want to add a site by domain so I control what is tracked.

Acceptance criteria

- Input may be `youtube.com`, `www.youtube.com`, or a full URL; store a normalized domain.
- Subdomain matching defaults on.
- Duplicates rejected.
- Add, edit, pause, delete without affecting other sites.
- Edit while enforcement is on requires the friend key.

### US-2.2 See what is being counted

As a user, I want to see the domain the monitor thinks I am on so I trust the timer.

Acceptance criteria

- Foreground watched surface shows domain + slice used + phase.
- Unwatched sites show as not monitored.

## Epic 3 — Per-site cycles

### US-3.1 Set limits per site

As a user, I want each site to have its own slice, cooldown, and max cycles.

Acceptance criteria

- Site A budget never consumes site B.
- Changing one policy does not change another.

### US-3.2 Count only active time

As a user, I want time to increase only while I am actually viewing that site.

Acceptance criteria

- Timer runs when a matching surface is foreground, screen on, and (if enabled) input within idle pause (default 60s).
- Timer pauses on app switch, unwatched URL, screen off, or lock.

### US-3.3 Resume the same slice

As a user, I want a two-minute check not to start a two-hour cooldown.

Acceptance criteria

- Leaving mid-slice pauses `slice_used_ms`.
- Returning continues the same slice until `slice_ms` is reached.

### US-3.4 Cycles, not one daily bucket

As a user, I want use → cooldown → use so I can open a site a few times a day without living in it.

Acceptance criteria

- Cycle counter increments when a slice is exhausted, not when the site is opened.
- Unused cycles do not roll over.

### US-3.5 Reset

As a user, I want a clear reset so yesterday does not burn today.

Acceptance criteria

- Counters reset at `reset_hour` local time (default 00:00).
- Cooldown that straddles reset is cleared by reset.

## Epic 4 — Redirect and cooldown

### US-4.1 Leave the site when the slice ends

As a user, when a slice is exhausted I want to be taken off that site immediately.

Acceptance criteria

- Within ~1–2 seconds: overlay plus browser navigation to home/blank for the current tab.
- Mapped apps: overlay until the user leaves.
- Other sites and other tabs should remain; minimum is current surface leaves the blocked site.

### US-4.2 Cooldown is a second timer

As a user, after a slice ends I want that site unusable for the cooldown duration.

Acceptance criteria

- Matching surfaces redirect / overlay for the whole cooldown.
- Other sites keep their own timers.
- UI shows slice used, cycles used, cooldown remaining.

### US-4.5 Last cycle ends the day

As a user, after I burn the last cycle I want no more slices until reset.

Acceptance criteria

- Phase becomes `blocked_until_reset`.
- Redirect / overlay keeps firing.
- Reset restores cycle index 0 and `in_slice`.

## Epic 5 — Friend key and hardening

### US-5.1 Enroll a trusted person

As a user, I want the unlock path bound to someone else so I cannot grant myself more time.

Acceptance criteria

- Setup requires a trusted E.164 number.
- Backend SMS the key to that number only.
- App UI, notifications, and on-device logs never show the key.
- Rebind number requires the current key.

### US-5.2 Override is rare and explicit

As a user who is locked out, I want one way out that I cannot perform alone.

Acceptance criteria

- Request unlock only notifies the friend.
- Grants: `extra_cycle`, `until_reset`, `pause_hours` (1 or 2).
- Every grant is an `UnlockEvent` the user can view but not delete without the key.
- Five failed key entries → backoff; service-number approve still works.

### US-5.3 Lock the escape hatches

As a user, I want turning the system off to be as hard as unlocking a site.

Acceptance criteria

- Device Owner: uninstall and clear-data denied.
- Accessibility lost: fail closed on watched sites until it returns.
- Soft mode without Device Owner shows a persistent bypass warning.
- Factory reset remains possible and is documented.

### US-5.4 Setup flow

As a user, I want a single checklist so enforcement is not half-on.

Acceptance criteria

1. Install APK, grant Usage / Accessibility / overlay / battery exemption.
2. Set Device Owner via ADB (hard mode).
3. Enroll friend number; server texts them the key.
4. Add policies (example: facebook.com 20m / 2h / 3).
5. Enforcement on. Policy or trusted-number changes require the key.
