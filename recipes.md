# Recipes

Four arrays people actually build, with
the choices that matter for each. All of
them are the guide's core config with
different settings, not new machinery.

## 1. The media library

The canonical case: films and shows on
three or four large disks, growing by
addition, watched often, changed rarely.

- One parity; two when the library passes
  the size where a rebuild would take a
  day (chapter 8's reasoning).
- Nightly automation per chapter 9, with
  the deletion threshold small — media
  libraries are deleted from by tired
  humans at midnight more than by
  hardware.
- `exclude` the download client's incoming
  folder until files are moved into the
  library proper: half-written files churn
  parity and protect nothing.

## 2. The photo and document archive

Smaller files, written once, *never*
re-obtainable — the stakes invert: the
array matters less than the backup.

- Two parity if the disks are old; the
  files are precious and the disks are
  the risk.
- Scrub on the full default rotation, and
  run a deliberate full `check` a few
  times a year; small archives make the
  full audit cheap.
- A real backup — a copy elsewhere, on a
  schedule — is not optional here.
  SnapRAID protects the shelf; the backup
  protects against the house. Chapter 13
  explains why these are different jobs.

## 3. SnapRAID + MergerFS, the classic pairing

SnapRAID protects disks but leaves you
several filesystems to browse. The
standard companion is **MergerFS**, which
presents several disks as one merged
volume without changing anything on them.

- MergerFS merges the *data* disks only.
  Parity and content stay outside it.
- Point your media server at the merged
  view; point SnapRAID at the real disks.
- The pairing is community-standard and
  documented in both projects' manuals;
  the lab did not run MergerFS, so treat
  the specifics there as documented-only
  and this chapter's SnapRAID side as the
  verified part.
- SnapRAID's own built-in `pool` command
  makes a lighter-weight merged view out
  of symlinks, for setups that do not
  want a filesystem layer; also
  documented-only here.

## 4. The shoebox of old disks

The disks you already own: 2 TB, 3 TB,
two 4 TBs, all different, none worth
matching into classic RAID.

- Parity disk = the **largest** (parity
  must cover the biggest data disk); the
  rest are data, however uneven.
- Expect the parity file to be sized by
  the largest data disk — the small disks'
  spare capacity is usable data space,
  not wasted.
- Old disks fail; that is the premise, not
  a surprise. Keep the deletion threshold
  strict, watch `smart` reports (below),
  and hold a second parity if three or
  more of the disks are veterans.

### A note on `smart`

SnapRAID's `smart` command reads the
disks' SMART self-monitoring data and
prints a health report with an estimated
probability of failure in the next year,
per disk. It needs real disks — the lab's
directories have no SMART to read — so it
is documented-only in this guide. On real
hardware it is the early-warning half of
the system: `scrub` finds damage that
happened; `smart` warns about the disk
likely to cause the next lot.

## The one-sentence version

Same program, same config shape: media
wants a strict deletion threshold, archives
want a backup beside the parity, browsers
want MergerFS in front, and the shoebox
wants the biggest disk on parity duty.
