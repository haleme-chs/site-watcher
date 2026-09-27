# Site Watcher — compact spec

Sideload, Device Owner optional-but-required for hard mode. Not for Play Store.

## 1. Actors

| Actor | Can do |
|---|---|
| User | Browse; see timers; request unlock (does not grant it) |
| Trusted friend | Receives key via server SMS; approves grants |
| App (Device Owner) | Enforce policies; refuse uninstall; lock policy changes |

## 2. Policy model

```
SitePolicy
  id
  domain              // registrable domain, lowercase
  match_subdomains    // default true
  mapped_app_ids[]    // package names
  slice_ms            // e.g. 20 min
  cooldown_ms         // e.g. 2 hours
  max_cycles          // e.g. 3
  enabled

GlobalConfig
  reset_hour                 // default 0 local
  idle_pause_ms              // default 60s
  block_unreadable_browsers  // default true
  trusted_phone_e164
  key_hash                   // hash of override key; plaintext never stored on device
  enrolled_at
  device_owner_active
```

## 3. Runtime state (per site, per day)

```
SiteDayState
  date
  cycle_index          // 0..max_cycles
  slice_used_ms
  phase                // in_slice | cooldown | blocked_until_reset
  phase_started_at
  cooldown_ends_at     // if phase == cooldown
```

Resume rule: leaving mid-slice pauses `slice_used_ms`. Returning continues the same slice until `slice_ms`, then cooldown and `cycle_index++`.

## 4. State machine

```
reset_hour
    → cycle_index = 0, slice_used_ms = 0, phase = in_slice

in_slice + matching foreground activity
    → add elapsed active ms
    → if slice_used_ms >= slice_ms
         cycle_index++
         if cycle_index >= max_cycles
             phase = blocked_until_reset
         else
             phase = cooldown
             cooldown_ends_at = now + cooldown_ms

cooldown + now >= cooldown_ends_at
    → phase = in_slice, slice_used_ms = 0

blocked_until_reset
    → stay until reset

friend override
    → apply grant, write UnlockEvent
```

Active time = watched surface foreground AND screen on AND last user input within `idle_pause_ms` (when idle pause is enabled).

## 5. Enforcement

When phase is `cooldown` or `blocked_until_reset` and the user is on a matching surface:

1. Full-screen overlay (Back does not dismiss).
2. Browsers: Accessibility navigates the current tab to that browser's home/blank.
3. Mapped apps: overlay until the user leaves; optional home intent.
4. Overlay: site, phase, remaining time. Primary action: Request unlock.

No local snooze, ignore today, or visible PIN.

## 6. Permissions and OS hooks

- `PACKAGE_USAGE_STATS`
- Accessibility service (URL, idle, navigate away)
- `SYSTEM_ALERT_WINDOW`
- Foreground service + ticker
- Battery optimization exemption
- Device Owner (`DevicePolicyManager`)
- Network: enroll / notify / confirm only

## 7. Device Owner setup

```text
adb install site-watcher.apk
adb shell dpm set-device-owner <package>/.admin.OwnerReceiver
```

Many OEMs require no existing device accounts. Without Owner the app is soft mode and must show a bypass warning.

v1 Owner policies: forbid uninstall; forbid clear-data. Optional: lock additional users / disable ADB after setup if the OEM allows.

Factory reset remains possible.

## 8. Friend key

### Enroll

1. User enters friend E.164 number.
2. Device `POST /enroll` with `device_id` + phone.
3. Backend generates a 10-byte key, grouped display form (e.g. `XK4F-29QM-7PLS`).
4. Backend stores `hash(key)` + device_id + phone. SMS from the service number to the friend only.
5. Device stores enrollment state, never the plaintext key.

### Request unlock

1. User chooses site + grant type.
2. Device `POST /unlock-request` → SMS to friend.
3. Nothing unlocks yet.

### Grants (v1)

- `extra_cycle` — decrement `cycle_index` (min 0), `phase = in_slice`, `slice_used_ms = 0`
- `until_reset` — skip enforcement for that site until next reset
- `pause_hours` — 1 or 2 hours, per site or global (specify in request)

### Apply

Friend texts the key to the service number, or dictates it once into a one-shot field that is not persisted.

Failed attempts: exponential backoff after 5 misses. Service-number approve still works.

```
UnlockEvent
  at, site_id, grant_type, requester = user, accepted = friend
```

## 9. Detection

- Browsers: Accessibility scrape of host on known packages.
- Apps: package in `mapped_app_ids` counts as that domain.
- Tick: 1s while a watched surface is foreground. Persist every 5s and on phase change (Room).

Preset map (v1, extendable):

| Domain | Packages (illustrative) |
|---|---|
| facebook.com | com.facebook.katana |
| instagram.com | com.instagram.android |
| youtube.com | com.google.android.youtube |
| reddit.com | com.reddit.frontpage |

## 10. Backend

- `POST /enroll` `{device_id, phone}` → SMS key, persist hash
- `POST /unlock-request` `{device_id, site, grant}` → SMS friend
- inbound SMS webhook or `POST /unlock-confirm` `{device_id, key, grant_id}`
- `POST /rebind` with current key

No usage telemetry. `device_id` is random, local.

## 11. UI

- Today: per site — phase, slice used / length, cycles used / max, cooldown remaining
- Policies: add domain, slice, cooldown, max cycles, mapped apps
- Trusted person: masked number, rotate key (requires current key)
- Unlock log
- Setup checklist: permissions + Device Owner + enroll

Staged policy edits that apply at next reset without a key: default **off**.

## 12. Threat model

| Bypass | v1 stance |
|---|---|
| Uninstall / clear data | Mitigated if Device Owner |
| Revoke Accessibility | Fail closed until restored |
| Chrome / official app | In scope |
| Extra browser | Parse or fail closed |
| User reads Sent SMS | Mitigated (server-sent SMS) |
| Change friend number | Requires key |
| Safe mode | Soft hole |
| Factory reset | Accepted |
| Friend shares key | Accepted |
| ADB after Owner | Harden if OEM allows |

## 13. v1 build order

1. Room models + state machine unit tests (no Android)
2. Foreground ticker + overlay + UsageStats package detect
3. Accessibility URL parsers for Firefox + Chrome
4. Domain → app map + app blocking
5. Device Owner receiver + uninstall lock
6. Backend enroll / unlock SMS
7. Policy UI + setup checklist
8. Remaining browsers + fail-closed unknown

## 14. Test script

1. Policy: `facebook.com`, 2 min slice, 5 min cooldown, max 2 cycles.
2. Open Facebook in Firefox; at 2:00 overlay + home; cooldown 5:00.
3. Open facebook.com in Chrome during cooldown → overlay, no slice burn.
4. Open Facebook app during cooldown → overlay.
5. After 5 min, 2 min more in Firefox → `blocked_until_reset`.
6. Request unlock `extra_cycle` → friend SMS; after grant, one slice available.
7. Key never appears on device UI or in the user's SMS app.
8. Uninstall denied when Device Owner is set.
