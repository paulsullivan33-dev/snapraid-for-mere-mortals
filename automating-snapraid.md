# Automating SnapRAID

SnapRAID has no daemon, so an array that
is only protected when someone remembers
to sync is protected by memory — the
least reliable component in any system.
This chapter turns chapters 5 and 6 into
a schedule. The lab verified every command
in the sequence interactively; the
scheduling itself is a pattern taught
from the project's documentation and
community practice, and is labelled as
such — no cron job ran in the sandbox.

## The safe order of operations

A scheduled run has three steps, and the
order is the safety:

1. **`diff` first, and read it.** Count the
   removals. If removals exceed a threshold
   you choose — a handful — **stop**. Do
   not sync. Something is wrong: a disk
   unmounted, a directory emptied by
   accident, a mount that failed at boot.
   Syncing now would protect the damage.
2. **`sync`.** Only after the diff looked
   like normal life: some additions, some
   updates, few or no removals.
3. **`scrub` on its rotation.** Independent
   of sync, on its own cadence — the
   default plan (8%, preferring blocks
   older than 10 days) finishes a full pass
   in about two weeks of daily runs.

The threshold in step 1 is the entire
point. Every horror story about parity
tools — "it protected the deletion" — is
a scheduled sync faithfully committing a
state nobody looked at. Chapter 7,
transcript t09 is the small print: delete,
sync, gone. Automation must look before
it leaps.

## The community script

Most home users do not hand-roll this.
The widely used **snapraid-aio-script**
(the "all-in-one" script maintained by the
community, at github.com/auanasgheps/
snapraid-aio-script) wraps exactly this
flow: it runs the diff, enforces a
deletion threshold you configure, syncs,
scrubs a slice, checks SMART health, and
emails or notifies you the result,
including — importantly — when it refused
to sync. If you adopt it, the two settings
to think about are the deletion threshold
(small; the default is conservative) and
the notification address (one you
actually read).

A hand-rolled equivalent in cron is a
dozen lines of shell around the three
commands above, with the diff's removal
count as the gate. Keep the gate.

## When to run it

- **Sync** after the array changes in a
  batch — for most homes, nightly is
  plenty; media libraries that grow on
  weekends can sync on weekdays' quiet.
- **Scrub** daily or a few times a week,
  at hours the machine is idle; it is
  read-only work and gentle at the default
  plan.
- **Never** overlap a sync with heavy
  writes to the array if you can avoid
  it: files changing mid-sync are re-read
  or deferred, and the run takes longer
  and reports more. Schedules exist
  precisely to pick quiet hours.

## Notifications are part of the system

An unattended SnapRAID should be boring
and slightly noisy: a short report after
each run ("synced, 3 added, 0 removed;
scrubbed 8%, no errors") builds the habit
of glancing, and makes the abnormal
report — the refusal, the first data
error — stand out the week it happens
rather than the year you go looking.
Silence is not success; silence is a
broken schedule. `status` by hand, once
in a while, is the audit of the
automation itself.

## The one-sentence version

Schedule it as diff-with-a-deletion-limit,
then sync, then scrub on rotation, with a
report you actually read — because the
only dangerous sync is the one nobody
looked at first.
