# More Parity

One parity protects against one failure
at a time. SnapRAID supports up to **six**
parity levels, each an independent
computation over the same stripes:

| Config line | Name in reports | Failures survived |
|---|---|---|
| `parity` | Parity (P) | 1 |
| `2-parity` | 2-Parity (Q) | 2 |
| `3-parity` … `6-parity` | R, S, T, Z | 3 … 6 |

"Failures" counts in units of damage: two
dead disks, or a dead disk plus a
scattered corruption elsewhere, or two
files' worth of bad blocks in the wrong
places. The lab ran one and two levels;
three and above are the same mechanism
with more files, and are documented-only
in this guide — the manual covers their
configuration, which differs only in the
extra lines.

## What an extra level costs and buys

Each level costs one more disk (or the
space of the largest data disk on one),
and more computation at sync time —
parity maths is fast, and on any modern
CPU the disks are the bottleneck, but the
parity files themselves are real terabytes
on a real array.

What it buys is not mainly "two disks
die at once", which is rare. It buys
surviving the **unlucky overlap**: a disk
dies, and during the long rebuild another
disk turns out to have a rotten patch —
the classic way single-parity arrays lose
data, because the rebuild reads every
surviving block and finds the ones that
were quietly bad. If your array is large,
your files irreplaceable, or your rebuild
window long, the second parity is cheap
insurance aimed at exactly that day.

## Adding a second parity to a running array

The lab did this for real (transcript
t11), and the procedure has one
non-obvious step: a plain `sync` **refuses**
when it finds a new parity file shorter
than the array expects — quite rightly,
because "parity disk not mounted" looks
exactly the same. It tells you so:

```
DANGER! One or more the parity files are smaller than expected!
It's possible that the parity disks are not mounted.
If instead you are adding a new 2-Parity level, initialize it with
'snapraid -d 2-parity fix', and then run 'snapraid sync'.
Alternatively, use 'snapraid --force-full sync' to rebuild all parity levels.
```

The lab took the second offered path:

```
$ snapraid -c snapraid-dual.conf --force-full sync
...
Everything OK
```

`--force-full` recomputes every parity
from the data — the expensive, certain
option. Afterwards both parity files stood
at the same size, and the test: corrupt a
file on d1 **and** wipe d2 at the same
time. `fix` recovered everything; every
checksum verified:

```
recovered photos/holiday-snaps.bin
recovered movies/birthday.mp4.bin
recovered movies/captions.txt
recovered music/concert.flac.bin
recovered music/new-track.flac.bin
       0 unrecoverable errors
```

## Split parity, briefly

A parity level can be split across
several smaller disks when no single disk
is big enough — `parity` accepts multiple
paths in sequence as the file grows, and
the manual documents the syntax. The lab
did not run a split; it changes capacity
planning, not concepts, and the refusal
messages in chapter 14 mention split
files because SnapRAID guards each piece
the same way.

## How many levels, honestly

- **One** — the home media array, files
  also backed up or re-obtainable. The
  common case, and a reasonable one.
- **Two** — large arrays, long rebuilds,
  or data you cannot get back another way.
- **Three or more** — specialist
  territory; at that point also ask
  whether a second copy of the data (a
  backup, chapter 13) is the better spend,
  because parity of any depth still has
  no history and no answer to deletion
  plus sync.

## The one-sentence version

Each extra parity level costs a disk's
worth of space and survives one more
simultaneous failure — and the second
level, aimed at damage discovered *during*
a rebuild, is the one most home arrays
that add any, add.
