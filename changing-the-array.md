# Changing the Array

Arrays change: a disk fills, a disk is
replaced before it fails, a bigger disk
arrives. This chapter covers the moves,
in the order you will meet them. Adding
parity was chapter 8; whole-disk
replacement after a failure was chapter
7's third rehearsal — what follows is the
planned version of these operations.

## Adding a data disk

1. Mount the new disk where it will live
   permanently.
2. Add a `data` line to the config with a
   fresh name, and a `content` line
   putting a copy on it.
3. Copy files onto it as you like.
4. `diff`, then `sync`.

No reformatting, no rebuild of the whole
array: the new disk's blocks join the
stripes from the next sync. One constraint
to plan around: if the new disk will hold
**more data than the parity disk can
cover** — parity must match the largest
data disk — deal with parity first
(next section).

## Replacing a data disk that still works

The calm version of chapter 7: copy is
safer than rebuild when the old disk can
still be read.

1. Mount the new disk alongside the old.
2. Copy everything across, preserving the
   directory layout (`rsync -a` or your
   file manager).
3. Point the config's `data` line for that
   name at the new mount point (and its
   `content` line likewise).
4. `diff` should show the files as equal —
   names, sizes and timestamps preserved,
   nothing to re-parity but bookkeeping.
5. `sync`, then retire the old disk.

If the copy scrambled timestamps, `diff`
shows updates instead; the sync that
follows re-hashes and re-parities those
files, which is slower but harmless.

## The parity disk is the ceiling

Parity covers the largest data disk. When
a data disk outgrows it:

- **Replace the parity disk with a bigger
  one**, mount it at the same path, and
  run a full sync to rebuild parity from
  the data (`--force-full sync` if the
  program asks, as it did in transcript
  t11). The data is untouched throughout;
  you are unprotected only for the
  duration of the rebuild, and only against
  failures in that window.
- Or **split the parity** across disks
  (chapter 8) if several smaller disks are
  what you have.

## Removing a disk

Empty it first (move its files to other
data disks and let a sync settle), then
remove its `data` and `content` lines and
sync again. Removing a disk that still
holds protected files and syncing is
chapter 7's deletion lesson at disk
scale: the files become unrecoverable
the moment the sync commits.

## Upgrading SnapRAID itself

The content and parity formats are stable
across versions, and an upgrade is:
install the new binary (chapter 3), run
`status` to confirm it reads the array,
and carry on. Do it deliberately — one
machine, not in the middle of a recovery —
and read the release notes for your jump;
the project documents any version with a
format note prominently, and a new
version's first act on an old array is a
read-only command in this guide's
discipline anyway.

## Retiring SnapRAID

Because nothing about your files is in a
SnapRAID format, leaving is: stop syncing,
delete the parity and content files, done.
Your disks are just disks. Few storage
systems let you walk away this cleanly;
it is worth knowing before you commit,
not after.

## The one-sentence version

Arrays grow one config line and one sync
at a time; the parity disk's size is the
ceiling on every data disk; and every
change follows the same liturgy — edit,
`diff`, look, `sync`.
