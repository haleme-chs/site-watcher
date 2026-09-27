# App page mockups

Simple phone frames for v1. Copy each box into Lucid as its own page or a row on the board.

Lucid board (source of truth if you paste these there):
https://lucid.app/lucidchart/fe86db04-eeb2-4381-b45f-42b37b4bfc08/edit?page=0_0

Screen list

1. Setup checklist
2. Today
3. Add / edit site
4. Block overlay
5. Request unlock
6. Unlock requested
7. Trusted person
8. Unlock log
9. Soft-mode warning (no Device Owner)

Bottom nav on main screens: Today | Sites | Friend | Log

---

## 1. Setup checklist

```
+-----------------------------+
|  Site Watcher               |
|  Setup                      |
|-----------------------------|
|  Finish these before        |
|  enforcement turns on.      |
|                             |
|  [x] Usage access           |
|  [x] Accessibility          |
|  [x] Display over apps      |
|  [x] Battery unrestricted   |
|  [ ] Device Owner (hard)    |
|  [ ] Trusted friend         |
|                             |
|  Soft mode: you can still   |
|  uninstall this app.        |
|                             |
|  [ Set Device Owner       ] |
|  [ Add trusted friend     ] |
|                             |
|  Enforcement: OFF           |
+-----------------------------+
```

## 2. Today

```
+-----------------------------+
|  Today              3:02 PM |
|-----------------------------|
|  facebook.com               |
|  IN SLICE                   |
|  8m / 20m used              |
|  Cycle 1 of 3               |
|  ################----       |
|                             |
|  youtube.com                |
|  COOLDOWN                   |
|  back in 1h 42m             |
|  Cycle 2 of 3               |
|                             |
|  reddit.com                 |
|  BLOCKED UNTIL RESET        |
|  3 of 3 cycles used         |
|  opens after 12:00 AM       |
|                             |
|  Watching: Chrome tab       |
|  facebook.com               |
|-----------------------------|
|  Today    Sites   Friend Log|
+-----------------------------+
```

## 3. Add / edit site

```
+-----------------------------+
|  <-  facebook.com           |
|-----------------------------|
|  Domain                     |
|  [ facebook.com           ] |
|  [x] Match subdomains       |
|                             |
|  Slice                      |
|  [ 20 ] minutes             |
|                             |
|  Cooldown                   |
|  [ 2  ] hours               |
|                             |
|  Max cycles / day           |
|  [ 3  ]                     |
|                             |
|  Mapped apps                |
|  [x] Facebook               |
|  [ ] Messenger              |
|  [ + Add package ]          |
|                             |
|  Enabled  [ ON ]            |
|                             |
|  Saving requires friend key |
|  if enforcement is on.      |
|                             |
|  [ Save ]        [ Delete ] |
+-----------------------------+
```

## 4. Block overlay

Full screen over browser or mapped app. Back does nothing.

```
+-----------------------------+
|                             |
|         facebook.com        |
|                             |
|         COOLDOWN            |
|         1h 42m left         |
|                             |
|  Slice used 20m of 20m      |
|  Cycle 2 of 3 today         |
|                             |
|  [ Request unlock ]         |
|                             |
|  This does not unlock.      |
|  Your friend gets a text.   |
|                             |
+-----------------------------+
```

Blocked-until-reset variant: title `BLOCKED UNTIL RESET`, subtitle `opens after 12:00 AM`.

## 5. Request unlock

```
+-----------------------------+
|  <-  Request unlock         |
|-----------------------------|
|  Site                       |
|  facebook.com               |
|                             |
|  Ask friend for             |
|                             |
|  ( ) One extra cycle        |
|  ( ) Until reset            |
|  ( ) Pause 1 hour           |
|  ( ) Pause 2 hours          |
|                             |
|  [ Send request ]           |
|                             |
|  Nothing changes until      |
|  they approve.              |
+-----------------------------+
```

## 6. Unlock requested

```
+-----------------------------+
|  Request sent               |
|-----------------------------|
|  Texted +1 ••• ••• 0147     |
|                             |
|  Waiting on your friend.    |
|                             |
|  facebook.com still in      |
|  cooldown.                  |
|                             |
|  [ Done ]                   |
+-----------------------------+
```

Optional one-shot key field (friend dictates; not saved):

```
|  Friend can read you the    |
|  key once.                  |
|  [ XXXX-XXXX-XXXX         ] |
|  [ Apply key ]              |
```

## 7. Trusted person

```
+-----------------------------+
|  Trusted person             |
|-----------------------------|
|  Number                     |
|  +••• ••• •• 0147            |
|  Enrolled Sep 27            |
|                             |
|  The key was texted to      |
|  them. It is not on this    |
|  phone.                     |
|                             |
|  [ Rotate key ]             |
|  [ Change number ]          |
|                             |
|  Both require the current   |
|  key.                       |
|-----------------------------|
|  Today    Sites   Friend Log|
+-----------------------------+
```

Enroll (first time):

```
+-----------------------------+
|  Add trusted friend         |
|-----------------------------|
|  Country + number           |
|  [ +1 ] [                 ] |
|                             |
|  They will get a text with  |
|  the override key. You will |
|  not see that key.          |
|                             |
|  [ Send key ]               |
+-----------------------------+
```

## 8. Unlock log

```
+-----------------------------+
|  Log                        |
|-----------------------------|
|  Today 3:11 PM              |
|  facebook.com               |
|  extra_cycle  granted       |
|                             |
|  Today 1:04 PM              |
|  youtube.com                |
|  request sent — no grant    |
|                             |
|  Yesterday 10:22 PM         |
|  reddit.com                 |
|  until_reset  granted       |
|                             |
|  Cannot delete entries      |
|  without the friend key.    |
|-----------------------------|
|  Today    Sites   Friend Log|
+-----------------------------+
```

## 9. Soft-mode banner

Shown on Today when Device Owner is not active.

```
+-----------------------------+
|  ! HARD MODE OFF            |
|  Uninstall and Settings     |
|  can still bypass this app. |
+-----------------------------+
```

## Lucid layout suggestion

Row 1: Setup | Today | Add site
Row 2: Overlay | Request unlock | Request sent
Row 3: Trusted person | Log | Soft-mode banner

Arrows

- Setup → Today when checklist complete
- Today site row → Add/edit site
- Overlay → Request unlock → Request sent → Today
- Friend tab → Trusted person
- Log tab → Unlock log
