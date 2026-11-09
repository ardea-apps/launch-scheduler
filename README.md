# Launch & Boat Dispatch Scheduler

A single-file, browser-based planning tool for community sailing programs that shuttle
passengers between a dock and a fleet of boats using a shared launch.

Given a handful of operating parameters, it builds a full day's dispatch schedule:
when the launch leaves the dock, which boat it's headed to, who it's dropping off and
picking up, and when each boat needs to be back at the rendezvous point.

No backend, no build step, no dependencies — it's one `.html` file that runs entirely
in the browser.

## Why this exists

Running a fleet off one launch is a small scheduling problem: the launch can only be
in one place at a time, boats return on their own clock, and a naive "first come,
first served" plan either strands late boats or leaves half the fleet idle at the
dock. This tool runs that scheduling problem for you and shows the result three ways
— a visual timeline, a round-by-round launch log, and a per-boat schedule — so both
the person planning the day and the person driving the launch can see the same plan.

## Features

- **Timeline view** — a Gantt-style chart with one lane per launch and one lane per
  boat, showing transit, sailing, and idle time at a glance.
- **Launch Rounds view** — the launch operator's own schedule: each round trip with
  dock departure, which boat, who's being dropped off vs. picked up, how long (if at
  all) that crew waited, dock arrival, and the time to tell the boat's skipper to be
  back for the next pickup.
- **By Boat view** — the same data grouped per boat, door-to-door for each passenger
  group (dock departure → sail departure → sail arrival → dock arrival).
- **Two priority strategies** for when the launch can't keep up with the whole fleet:
  - *Minimize on-water wait* — never delays a boat that's already sailing in order to
    start a new one; some boats may go unused if the launch can't cover them.
  - *Launch every boat* — spreads the launch's capacity across the whole fleet,
    accepting more waiting per boat in exchange for using every boat.
- **Capacity awareness** — automatically detects when the fleet's demand exceeds
  what the launch(es) can supply, and explains the trade-off rather than silently
  producing an impossible schedule.
- **Print / Save as PDF** — a clean, self-contained printout of the launch log for
  handing to the operator on paper.
- **Per-event shareable links** — bake a specific event's settings into the URL
  (see below) so a bookmark opens straight to that day's schedule.
- **Multi-launch support** — model more than one launch if the program runs more
  than one.

## Quick start

Open `launch_scheduler.html` in any modern browser — locally, or hosted anywhere
that can serve a static file (no server-side logic required).

1. Fill in the parameters (start time, number of boats, trip duration, launch leg
   duration, departure rounding, passengers per boat, operating window, number of
   launches, priority strategy).
2. Click **Generate schedule**.
3. Switch between **Timeline**, **Launch Rounds**, and **By Boat** to view the
   result different ways.
4. Use **Print / Save as PDF** on the Launch Rounds view for a paper copy.

## Parameters reference

| Field | Meaning |
|---|---|
| Start time | When the launch begins its first run of the day |
| Number of boats | Size of the fleet in rotation |
| Trip duration | How long each boat's sailing trip lasts |
| Launch trip, one way | Time for the launch to go dock → boat (or boat → dock) |
| Departure rounding | Dock departures round up to the nearest multiple of this many minutes |
| Passengers per boat | Fixed capacity per trip, used for the passenger totals |
| Operate until | No new voyages start after this time; anyone already at sea is still retrieved |
| Number of launches | How many independent launches are running (default 1) |
| Priority when tight | *Minimize on-water wait* vs. *Launch every boat when possible* — see Features above |

## How the schedule is built

The launch is modeled as a shared, single-threaded resource. Each boat's first trip
gets a "virtual" ideal dispatch slot spaced two launch-legs apart, so first trips
compete fairly for launch time against later pickups. From then on, every boat's
return creates a real pickup requirement — ideally the launch arrives at the
rendezvous exactly as the boat does, drops the next group, and brings the previous
group home in the same run.

When two requirements collide (because there's only one launch, or more boats than
it can serve), the active **priority strategy** decides who goes first. The engine
always finalizes the actual retrieval time of a boat's *last* group of the day as a
dedicated "closing pickup," since that's a real launch trip too — it isn't left
implicit or missing from the schedule.

This is a greedy heuristic, not a global optimizer: it makes the locally-best choice
at each decision point rather than searching all possible schedules. In practice this
produces the same steady, sensible rhythm a real launch operator would settle into.

## Per-event shareable links

Rather than re-entering the same numbers every time, set up the parameters for a
specific recurring event, click **🔗 Copy link for this event**, then bookmark the
tab (Ctrl+D / Cmd+D). The event's settings are encoded directly in the URL, e.g.:

```
launch_scheduler.html?start=11:00&end=15:00&boats=3&trip=50&launch=10&round=5&pax=6&launches=1&priority=minimizeWait
```

Opening a link like this auto-fills every field and generates the schedule
immediately — no re-entry needed. Keep one bookmark per event type (e.g. "Community
Sailing Day," "Regatta Day") with that event's fixed parameters.

## Hosting

It's a single static file with no server logic: upload it via FTP to
`bluehorizonsailing.org/tools/launch-scheduler/` or host it anywhere else.

## Browser support

Uses standard modern browser APIs: `URLSearchParams`, the Clipboard API (for the
"copy link" button — falls back gracefully if unavailable), `history.pushState`
(for updating the address bar without a reload), and native `window.print()` for
the PDF/print flow. Works in current versions of Chrome, Firefox, Edge, and Safari.

## Known limitations

- Assumes every boat has the same passenger capacity and every trip is the same
  duration — no per-boat or per-trip customization.
- The priority heuristic is greedy, not globally optimal; it won't necessarily find
  the mathematically best possible schedule when the fleet is heavily oversubscribed.
- No persistence beyond what's encoded in the URL — refreshing without event
  parameters resets to the built-in defaults.

## Changelog

- **v1.0** — Initial release: schedule generation, Timeline / Launch Rounds / By Boat
  views, capacity detection, print/PDF export, multi-launch support, priority
  strategies, per-event shareable links.
