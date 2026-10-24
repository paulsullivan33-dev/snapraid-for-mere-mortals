# What SnapRAID Is

SnapRAID is a program that protects a set
of disks — an *array* — by keeping
**parity**: recovery information computed
from the data on the other disks. If a disk
dies, or a file is corrupted, or a file is
deleted by mistake, SnapRAID can rebuild
what was lost from the parity and the
disks that survive. It is free, open-source
software (GPL), written by Andrea
Mazzoleni, and it runs on Linux and
Windows.

What makes it different from the RAID most
people picture is *when* it works. RAID
writes parity continuously, as you write
files. SnapRAID does nothing until you tell
it to **sync** — then it reads the array,
notices what changed since last time, and
updates the parity to match. Between syncs
it is not running at all. Everything
SnapRAID does follows from that one
choice, so hold it in mind: **SnapRAID
protects the array as it was at the last
sync.**

## The shape: data disks and parity disks

An array has two kinds of members:

| Member | Holds | Notes |
|---|---|---|
| **Data disk** | Your files, as usual | Any size, already full is fine |
| **Parity disk** | One parity file | Must fit the largest data disk's contents |

One parity disk lets you survive any *one*
failure at a time: one dead disk, or one
disk's worth of scattered damage. A second
parity — chapter 8 — survives two, and
SnapRAID supports up to six levels. The
data disks stay ordinary disks with
ordinary files on ordinary filesystems the
whole time. You can browse them, copy from
them, and stop using SnapRAID tomorrow
without reformatting anything. There is no
lock-in because there is no special format:
the only new things on disk are the parity
file and a small index called the
**content file** (chapter 2).

## Who it is for

SnapRAID's own description says it is
"mainly targeted for a home media center,
where you have a lot of big files that
rarely change" — and that targeting is the
whole design. It suits:

- a media library that grows by addition
  and is otherwise only read;
- photo and document archives, written once
  and kept;
- a shoebox of mismatched disks collected
  over years, too uneven for classic RAID;
- anyone who wants a dead disk to be an
  inconvenience rather than a loss, without
  buying matched hardware.

The common thread is files that are
**large and mostly static**. A disk of
small files that change every hour is the
opposite of the target, and chapter 13
says why that hurts.

## What it is not

- **Not a backup.** Parity rebuilds what
  the array held at the last sync; it has
  no history, no versions, and no answer
  to fire, theft, ransomware or deleting a
  file and then syncing. Chapter 13 is
  blunt about this. You still need backups
  of anything irreplaceable.
- **Not real-time RAID.** Nothing is
  protected until the next sync runs. New
  files written an hour ago are invisible
  to the parity.
- **Not a filesystem or a pool.** By
  itself SnapRAID does not merge your
  disks into one volume; it only protects
  them. (A built-in `pool` option makes a
  view of symlinks, and most people pair
  it with MergerFS instead — chapter 11.)
- **Not always-on software.** It runs when
  invoked — by you, or by a schedule you
  set up (chapter 9) — and otherwise does
  not exist. There is no daemon to crash,
  and no protection happening in the
  background either.

## The one-sentence version

If your files live on a few big disks that
mostly sit still, SnapRAID lets any one of
them die — or quietly rot, or suffer a
mistaken deletion — and gives you the
contents back, at the price of one extra
disk and remembering to sync.
