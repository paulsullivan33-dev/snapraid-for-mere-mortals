# SnapRAID for Mere Mortals

Most home servers hold the same kind of
thing: a few large disks full of files
that barely ever change — a media library,
a photo archive, recordings, downloads you
mean to keep. Losing one disk means losing
everything on it, but traditional RAID is
an awkward answer: it wants matched disks,
runs all of them all the time, and rebuilds
the whole array or nothing. **SnapRAID** is
the open-source answer built for exactly
this shape of problem. It takes a snapshot
of parity — recovery information — over
disks you already have, of whatever sizes,
filled with files already, and from that
snapshot it can rebuild a dead disk, repair
a silently corrupted file, or bring back a
file you deleted by mistake. This guide
explains how it works, walks through
setting it up by hand, and rehearses the
recoveries on a real array before you ever
need them for real.

The lab behind this guide ran the real
thing: **SnapRAID 15.0**, the project's
current release on build day, compiled
from the release source in this sandbox
and pointed at a small array of two data
disks and one parity disk, all of them
directories standing in for disks —
SnapRAID checks that its disks really are
separate devices, and the one declared
deviation of this lab is a test flag that
tells it not to, stated in full in chapter
3 and in `criteria.md`. Onto the array went
a handful of invented files with their
checksums recorded, and then the system was
exercised end to end: a first sync, a
status report, a diff, a full scrub and a
full check, a file corrupted in place and
repaired from parity, a file deleted and
brought back, a file deleted *and synced
over* and shown to be gone for good, a
whole disk wiped and rebuilt, and a second
parity added the way the program insists
on, after which a corruption and a disk
loss at the same time were both repaired.
What the lab could not do, it did not
fake: directories are not disks, so the
hardware-facing features — SMART reports,
spin-up probes, temperature limits — are
taught from the project's own manual and
labelled as such, and so is everything
about Windows, where the lab never ran.

## What this guide covers

- [**What SnapRAID is**](what-snapraid-is.md) —
  the mental model: snapshot parity rather
  than real-time RAID, aimed at big files
  that rarely change; what it is not (a
  backup, a filesystem, a RAID controller)
- [**The moving parts**](the-moving-parts.md) —
  the parity file, the content file, blocks
  and hashes: what each holds, and what
  breaks when each is lost
- [**Installing SnapRAID**](installing-snapraid.md) —
  the package, the source build the lab
  performed, Windows in brief, and the
  lab's declared deviation, told in full
- [**Writing the config**](writing-the-config.md) —
  `snapraid.conf` line by line: parity,
  content, data, excludes, and the rules
  SnapRAID enforces about where things live
- [**Your first sync**](your-first-sync.md) —
  what `sync` actually does, reading
  `status`, and using `diff` as the
  pre-flight check before every sync
- [**Checking and scrubbing**](checking-and-scrubbing.md) —
  `check` proves everything now; `scrub`
  proves a slice at a time and remembers
  which; how the two divide the work
- [**Recovering files**](recovering-files.md) —
  `fix` for the three failures that matter:
  a corrupted file, a deleted file, and a
  whole disk — each rehearsed in the lab
- [**More parity**](more-parity.md) —
  a second parity and beyond: what extra
  levels buy, what they cost, and how to
  add one to a running array
- [**Automating SnapRAID**](automating-snapraid.md) —
  running it on a schedule without hurting
  yourself: the order of operations, the
  deletion threshold, and the community
  script that wraps it all
- [**Changing the array**](changing-the-array.md) —
  adding a disk, replacing a disk, growing
  past your parity, and upgrading the
  program itself
- [**Recipes**](recipes.md) — a media
  library, a photo archive that never
  changes, the classic MergerFS pairing,
  and a mixed shoebox of old disks
- [**The neighbours**](the-neighbours.md) —
  hardware and software RAID, ZFS, Unraid,
  Btrfs snapshots and plain backup: what
  each one trades
- [**Limits**](limits.md) — not a backup,
  not real-time, blind between syncs, and
  the other limits stated plainly
- [**Troubleshooting**](troubleshooting.md) —
  the same-device refusal, the parity that
  is smaller than expected, the file that
  cannot be recovered, and the other traps
  this guide's lab fell into so you don't
  have to
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Building the machine itself — choosing
disks, controllers and cases is a subject
of its own — and filesystems in depth:
SnapRAID sits on top of whatever filesystem
you already use and this guide does not
teach one. Data recovery for hire, arrays
bigger than a home server, and the
mathematics of Reed–Solomon codes beyond
what chapter 2 needs are likewise pointed
at, not taught.

## How to use this guide

Read chapter 1 for the model and chapter 2
before configuring anything — knowing which
file holds what turns later confusion into
a named problem. Chapter 4 writes a config
you can rehearse with on spare directories
before trusting it with real disks; do the
recovery chapter's exercises on that same
practice array, because the first time you
run `fix` should not be the day a disk
dies. Chapter 9 is not the optional ending:
an array that is never synced protects
nothing, so the schedule is part of the
setup. The lab's transcripts live in
`snapraid-lab/transcripts/`; its notes are
`examples/lab-notes.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [MergerFS for Mere Mortals](https://paulsullivan33-dev.github.io/mergerfs-for-mere-mortals/)
- [ZFS for Mere Mortals](https://paulsullivan33-dev.github.io/zfs-for-mere-mortals/)
- [Backups for Mere Mortals](https://paulsullivan33-dev.github.io/backups-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/snapraid-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/snapraid-for-mere-mortals/blob/main/LICENSE-CODE).
