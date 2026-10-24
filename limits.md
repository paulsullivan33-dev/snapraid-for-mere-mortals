# Limits

The standing rule of this series: the
limits in plain sight. SnapRAID's follow
directly from its one design decision —
snapshot parity — plus a few from being a
small, single-purpose tool.

## Not a backup

Say it until it is boring. SnapRAID has
no history: one protected state, the last
sync. It cannot answer "the file as it
was last month", "the folder I deleted
last week and synced over", fire, theft,
flood, or malware that encrypts the array
and gets synced. Files you cannot
re-obtain need a backup — a separate copy,
ideally elsewhere — whether or not
SnapRAID is also protecting the shelf.

## Blind between syncs

New and changed files are unprotected
until the next sync, and a disk that dies
in that window takes the unsynced changes
with it. Worse, a file *corrupted* in that
window can be synced as-is if the
corruption also touched size or timestamp
in a way that looks like an edit —
`check` and `scrub` are the counterweight,
and they only run when scheduled.

## Deletion is protected only until it isn't

Chapter 7's law: an accidental deletion
is recoverable up to the next sync, and
permanently gone after it. Every
automation decision in chapter 9 exists
because of this limit. There is no
undelete fairy beyond that line.

## Built for big, still files

Small files that change constantly are
the worst fit: sync re-reads and
re-parities churn, the content file grows
with file count, and the protection gap
between syncs covers most of the file's
life. A home directory, a mail spool, a
database — protect those with backups and
snapshots, not SnapRAID.

## Whole disks at a time, single machine

- A failure bigger than your parity count
  loses the failed disks' files (only
  those — survivors are untouched, but
  the loss is total for what failed).
- The array lives on one machine. It does
  not replicate, fail over, or serve
  anything; it is protection, not
  availability. A dead machine means a
  working array you cannot reach until
  the machine is fixed.
- Recovery speed is bounded by reading
  the surviving disks end to end. On large
  disks that is most of a day, during
  which (on single parity) one more bad
  block is a loss. Chapter 8's second
  parity is priced against this window.

## It trusts its config and its operator

SnapRAID refuses arrays it can prove are
wrong (chapter 4), but it cannot prove a
mount is the *right* disk, that a sync is
well-timed, or that you read the diff. It
is a sharp tool with good guards, worked
by hand or by a schedule a human set up.
The guards stop the known traps; the
operator is the guard for the rest.

## The one-sentence version

SnapRAID gives a few big quiet disks an
excellent answer to "one of them died or
rotted" — and no answer at all to
history, disaster, or constant change, so
pair it with a backup and a schedule and
know which job is whose.
