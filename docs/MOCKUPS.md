# App page mockups

Wireframes match the Lucid board (9 screens, 3×3).

Lucid: https://lucid.app/lucidchart/fe86db04-eeb2-4381-b45f-42b37b4bfc08/edit?page=0_0

## Flow

```mermaid
flowchart LR
  subgraph row1 [" "]
    direction LR
    setup["Site Watcher · Setup\n\nFinish setup before enforcement turns on.\n✓ Usage access\n✓ Accessibility\n✓ Display over apps\n✓ Battery unrestricted\n☐ Device Owner hard\n☐ Trusted friend\n\nSOFT MODE\nYou can still uninstall this app.\n\nSet Device Owner\nAdd trusted friend\n\nEnforcement: OFF"]
    today["Today · 3:02 PM\n\nfacebook.com\nIN SLICE · 8m / 20m used\nCycle 1 of 3\n\nyoutube.com\nCOOLDOWN · back in 1h 42m\nCycle 2 of 3\n\nreddit.com\nBLOCKED UNTIL RESET\n3 of 3 cycles used\nopens after 12:00 AM\n\nWatching: Chrome tab\nfacebook.com\n\nToday · Sites · Friend · Log"]
    edit["Edit site\n\nDOMAIN facebook.com\n✓ Match subdomains\n\nSLICE 20 minutes\nCOOLDOWN 2 hours\nMAX CYCLES 3\n\nMAPPED APPS\n✓ Facebook\n☐ Messenger\n+ Add package\n\nENABLED ON\n\nSave · Delete"]
  end

  subgraph row2 [" "]
    direction LR
    overlay["Blocked overlay\n\nfacebook.com\n\nCOOLDOWN\n1h 42m left\n\nSlice used 20m of 20m\nCycle 2 of 3 today\n\nRequest unlock\n\nThis does not unlock.\nYour friend gets a text."]
    req["Request unlock\n\nSite facebook.com\n\nASK FRIEND FOR\n○ One extra cycle\n○ Until reset\n○ Pause 1 hour\n○ Pause 2 hours\n\nSend request\n\nNothing changes until they approve."]
    sent["Request sent\n\nTexted +1 ••• ••• 0147\n\nWaiting on your friend.\n\nfacebook.com still in cooldown.\n\nDone\n\nOne-shot key\nXXXX-XXXX-XXXX\nApply key"]
  end

  subgraph row3 [" "]
    direction LR
    friend["Trusted person\n\nNUMBER +1 ••• ••• 0147\nEnrolled Sep 27\n\nThe key was texted to them.\nIt is not on this phone.\n\nRotate key\nChange number\n\nBoth require the current key.\n\nToday · Sites · Friend · Log"]
    log["Unlock log\n\nToday 3:11 PM\nfacebook.com\nextra_cycle granted\n\nToday 1:04 PM\nyoutube.com\nrequest sent — no grant\n\nYesterday 10:22 PM\nreddit.com\nuntil_reset granted\n\nCannot delete entries\nwithout the friend key.\n\nToday · Sites · Friend · Log"]
    soft["HARD MODE OFF\n\nUninstall and Settings\ncan still bypass this app.\n\nWHAT TO DO\nSet Device Owner to enable\nhard mode.\n\nShown on Today when Device\nOwner is not active."]
  end

  setup -->|setup complete| today
  today -->|edit site| edit
  overlay -->|request| req
  req -->|sent| sent
  sent -->|done| today
  overlay -->|friend| friend
  today -->|log| log
  today -.->|banner| soft
```

## Navigation

| From | Action | To |
|---|---|---|
| Setup | setup complete | Today |
| Today | edit site | Edit site |
| Today | Friend tab | Trusted person |
| Today | Log tab | Unlock log |
| Today | Device Owner missing | Hard mode off banner |
| Blocked overlay | Request unlock | Request unlock |
| Request unlock | Send request | Request sent |
| Request sent | Done | Today |

Bottom nav on Today, Trusted person, and Unlock log: **Today · Sites · Friend · Log**.

## Screen copy

Same content as the Lucid frames, kept here so the diagram and the board stay in sync.

### Setup

Finish setup before enforcement turns on.

- Usage access, Accessibility, Display over apps, Battery unrestricted: done
- Device Owner (hard), Trusted friend: not done
- Soft mode: you can still uninstall this app
- Actions: Set Device Owner, Add trusted friend
- Enforcement: OFF

### Today

- facebook.com — IN SLICE, 8m / 20m, cycle 1 of 3
- youtube.com — COOLDOWN, back in 1h 42m, cycle 2 of 3
- reddit.com — BLOCKED UNTIL RESET, 3 of 3, opens after 12:00 AM
- Watching: Chrome tab, facebook.com

### Edit site

facebook.com, match subdomains, slice 20m, cooldown 2h, max cycles 3, Facebook app mapped, enabled ON, Save / Delete.

### Blocked overlay

facebook.com cooldown, 1h 42m left, slice 20m of 20m, cycle 2 of 3. Request unlock does not unlock; friend gets a text.

### Request unlock

Grants: one extra cycle, until reset, pause 1 hour, pause 2 hours. Nothing changes until they approve.

### Request sent

Texted masked number. Site still in cooldown. Optional one-shot key field, not persisted.

### Trusted person

Masked number, enrolled date. Rotate key / change number both require the current key.

### Unlock log

Append-only. Cannot delete without the friend key.

### Hard mode off

Banner on Today when Device Owner is not active.
