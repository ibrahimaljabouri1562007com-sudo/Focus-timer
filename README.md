# FOCUS TIMER

### ▶ https://ibrahimaljabouri1562007com-sudo.github.io/Focus-timer/

**That link is the app.** Same URL on the phone and at the desk — open it anywhere.

A locked 50/5 cadence timer. Plan once, then it runs itself until the mission is over.
Black · silver · red. No pause, no stop, no reset.

---

## Launch

**Desk** — double-click **`FOCUS.cmd`**. It opens the hosted link in a chrome-less Chrome
app window, so the countdown shows in the **window title and the taskbar** even when the
window is shrunk down beside the chat. No local server is involved any more.

**Phone** — open the link and use *Add to Home Screen*. It launches fullscreen with its
own icon.

> The app is served from GitHub Pages out of the `main` branch of
> [`Focus-timer`](https://github.com/ibrahimaljabouri1562007com-sudo/Focus-timer).
> `FOCUS.cmd` is deliberately **not** in that repo — it holds a local Windows path.

---

## Objectives

A list you fill ahead of time, so sitting down to work is a choice between prepared
options instead of a blank field.

- **Add** — type a name, Enter. Name only; the duration is chosen fresh each session.
- **Pick** — tap a pending objective → the plan screen opens with the mission name
  already filled and the cursor in the duration field. Type a number, engage.
- **Verdict** — when an objective-backed mission ends, the debrief asks
  **MISSION ACCOMPLISHED ?** — YES or NO, nothing in between. The answer is written
  back to the list as ✓ or ✗.
- **The question waits.** It survives closing the app. Walk away mid-mission, come back
  tomorrow, and the question is sitting there before you can do anything else.
- **Settled objectives sink** below a divider, dimmed, and are **outside the greeting
  rule** — only unattended ones decide what you see on launch.
- **No retry.** Delete a settled objective and add it again to redo it.
- **Free mission** — start one without an objective and no question is asked; there is
  nothing to mark.

**Launch rule:** any *pending* objectives → the objectives list is the first screen.
None → straight to the plan screen.

### Sync

Sign in once per device (the **SIGN IN TO SYNC** link on the objectives screen) and the
list follows you between phone and desk.

- **Local-first.** Reads answer from a local mirror, so the list paints instantly and
  still works with no signal. Writes land locally, then queue in an outbox that flushes
  when it can — a dropped connection can never lose a verdict.
- **The timer never touches the network.** If Supabase or the CDN is unreachable the
  app falls back to local-only and the cadence runs exactly the same. Losing WiFi is
  not an escape hatch.
- **On launch** the app waits up to 2.5s for a sync before choosing which screen to
  show, so objectives added on the phone decide what greets you at the desk.
- Objectives written while signed out are carried up to the account on first sign-in.

**Storage:** Postgres table `focus_objectives` in Supabase, guarded by row level
security (`auth.uid() = user_id`) — every row is scoped to its owner. The publishable
key in `index.html` is public **by design**: signed out, the API returns zero rows and
refuses writes. The local mirror lives in `localStorage` under
`ioi.focus.objectives.v1`, with the outbox at `ioi.focus.outbox.v1`.

---

## How it runs

1. **Plan** — name the mission, then **type** how many 50s into the `N.50` field
   (1–12; `+` / `−` and the arrow keys work too), press **ENGAGE**. That press is the
   only input the timer ever asks for.
2. **Run** — `W1.50` → 50 min → *gentle falling chime* → 5 min recovery → *crisp rising
   chime* → `W2.50` → … Each work block is red; recovery turns the whole interface silver.
   The screen holds four things only: mission number, mission name, the countdown, and
   the block tag (`W1.50`) beneath it. Nothing else renders while a mission runs.
3. **Debrief** — after the **last 50** (a mission always ends on work, never on a break)
   a heavier signal plays, the screen reads **MISSION 0N TIME IS OVER**, and the timer is
   spent. Two ways out: **NEW MISSION** or **CLOSE**.

`3.50` = 50 / 5 / 50 / 5 / 50 → 2h 40m, two breaks.

## The lock

- The run screen contains **zero buttons**. There is nothing to press, by construction.
- The clock is **timestamp-driven**, not a counter. Closing the tab, reloading, or
  sleeping the machine buys back nothing — reopen and it has kept running without you.
- A resumed session lands silently: alarms you missed while away do **not** replay.
- Closing the window during a work block raises the browser's "leave site?" warning.

## Sound

Three synthesized signals, no audio files, nothing to download:

| moment | character | length |
|---|---|---|
| 50 done → recovery | gentle, smooth, falling | ~1.5s |
| recovery done → next 50 | crisp, rising, two pulses | ~0.7s |
| mission over | heavier, low bed, resolving | ~2.3s |

Audio unlocks on the ENGAGE press (browsers require a gesture) — so always start a
mission from the button, never by hand-editing storage.

---

## Testing knobs (dev only)

Append to the URL to compress the cadence and watch a whole mission in seconds:

```
?t=8        work blocks become 8s (breaks auto-scale)
?t=60&b=30  work 60s, break 30s
```

Real use has no query string: 50 min / 5 min.

## Reset

The mission number and any in-flight mission live in `localStorage`
(`ioi.focus.v2`, `ioi.focus.count.v2`). To start the numbering over, open DevTools on the
timer window and run:

```js
localStorage.clear(); location.reload();
```

---

## Files

| file | role |
|---|---|
| `index.html` | the whole timer — markup, theme, engine, synthesized audio. No dependencies. |
| `FOCUS.cmd` | one-click launcher: private server + app-mode window. |

The only network request is the Google Fonts stylesheet (Barlow / Barlow Condensed).
Offline it falls back to a condensed system stack and everything still works.
