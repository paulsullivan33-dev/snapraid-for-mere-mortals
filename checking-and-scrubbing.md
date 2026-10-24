# Checking and Scrubbing

Sync keeps parity current, but nothing so
far has *proved* the data on disk still
matches the hashes — that a file hasn't
quietly rotted, or a controller hasn't
been writing garbage. Two commands do the
proving, and they divide the work by cost.
The lab ran both (transcripts t05, t06).

## `check` — prove everything, now

```
$ snapraid -c snapraid.conf check
...
100% completed, 24 MB accessed in 0:00
Everything OK
```

`check` re-reads every file, re-hashes
every block, and verifies the parity
against the data. It is the full audit. On
a clean array it ends "Everything OK",
exit code 0. On the lab's corrupted array
(chapter 7, transcript t07) it ended very
differently:

```
DANGER! Unexpected silent data errors!
       1 data errors
```
(exit code 1, and the file named in the
full output.)

The cost is reading the whole array, which
on a multi-terabyte media server is hours
of every disk spinning. That cost is why
`check` is the exception, run when you
suspect something — and why `scrub`
exists for routine.

## `scrub` — prove a slice at a time, and remember

`scrub` does the same hash verification,
but on a **plan**: a percentage of the
array, choosing the blocks that have gone
longest without verification. Every block
remembers when it was last scrubbed, and
`status` draws the picture:

```
The oldest block was scrubbed 0 days ago, the median 0, the newest 0.
100% of the array is not scrubbed.   (before any scrub)
```

The default plan scrubs **8%** of the
array, preferring blocks not scrubbed in
the last 10 days — run on a schedule, that
verifies the whole array roughly every
couple of weeks while never reading more
than a slice at a time. The lab overrode
the plan to prove the entire small array
at once:

```
$ snapraid -c snapraid.conf scrub -p 100 -o 0
...
Everything OK
```

`-p 100` is the percentage; `-o 0` makes
every block eligible regardless of age.
On a real array, leave the defaults alone
and let the rotation do its work.

Scrub also has a second job: blocks it
finds already damaged are marked so that a
later `fix` (or `check`) knows the damage
predates any recent change, and scrubbing
parity blocks refreshes parity stored on
media that itself can rot.

## Which one, when

| Situation | Command |
|---|---|
| Routine, scheduled | `scrub` (default plan) |
| After a scare: power cut, disk warnings, a moved machine | `check` |
| Before trusting a recovery | `check` |
| First week with a new array | one full `scrub -p 100 -o 0`, then relax |

## The honest catch

Verification only helps if it runs.
SnapRAID has no daemon (chapter 1), so
nothing scrubs until you or your schedule
ask. A corrupted file sits corrupted —
recoverable, because parity still holds
the good blocks — until a scrub or check
notices, and it stays recoverable only
while the damage stays within your parity
count. Chapter 9's schedule pairs the two
habits: sync after changes, scrub on a
rotation, so detection never depends on
memory.

## The one-sentence version

`check` is the full audit you run on
suspicion; `scrub` is the same audit in
slices, remembering which slice it did, so
a schedule can keep the whole array
continuously proven without ever reading
all of it at once.
