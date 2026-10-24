# Recovering Files

Everything so far was preparation. This
chapter is the payoff: `fix`, the command
that rebuilds what was lost from parity.
The lab rehearsed the three failures that
actually happen, and verified every
recovery against the checksums recorded
beforehand. Rehearse them the same way on
your own practice array — the first `fix`
you ever run should not be on a bad day.

## Failure 1: a file silently corrupted

A few bytes in the middle of a file were
overwritten in place — the stand-in for
bit rot, a failing disk writing garbage,
a bad cable. The file still exists, same
size, same name. Only its hash knows.

```
$ snapraid -c snapraid.conf check
DANGER! Unexpected silent data errors!
       1 data errors          (exit code 1)

$ snapraid -c snapraid.conf fix
recovered photos/family-video.bin
       1 recovered errors

$ sha256sum -c checksums-before.txt
disk1/photos/family-video.bin: OK
```
(Transcript t07.)

`fix` re-reads the stripe, finds the block
whose hash disagrees, rebuilds it from
the parity and the surviving disks, and
writes it back. The checksum returning to
OK is the proof: the file is byte-for-byte
what it was at the last sync.

## Failure 2: a file deleted by mistake

```
$ rm disk2/music/concert.flac.bin
$ snapraid -c snapraid.conf fix
recovered music/concert.flac.bin
$ sha256sum -c checksums-before.txt
disk2/music/concert.flac.bin: OK
```
(Transcript t08.)

The content file still lists the file and
its hashes; the parity still holds its
blocks; so `fix` rebuilds it whole. The
condition is absolute and worth memorising:

> **A deleted file is recoverable only
> until the next sync.**

The lab proved the other side too
(transcript t09): delete a file, run
`sync` — which quite correctly updates the
array to "the file is gone" — and `fix`
afterwards finds nothing to recover. The
deletion has become the protected truth.
This is the single most important
operational fact in this guide, and it is
why chapter 9's scheduled syncs refuse to
proceed when deletions pass a threshold.

## Failure 3: a whole disk dies

The lab emptied a data disk entirely and
left an empty directory at its path —
the shape of a dead disk replaced by a
fresh one:

```
$ snapraid -c snapraid.conf fix
recovered movies/birthday.mp4.bin
recovered music/concert.flac.bin
recovered music/new-track.flac.bin
      51 recovered errors
       0 unrecoverable errors

$ sha256sum -c checksums-before.txt
(all d2 files): OK
```
(Transcript t10.)

Notes from the run:

- The dead disk's **content copy died with
  it**; SnapRAID loaded another copy and
  carried on, exactly as chapter 2 said.
- Recovery needs the disk's path to exist
  (an empty replacement disk mounted in
  the same place). A missing path is an
  error at startup, not a recovery.
- `fix -d d2` limits a fix to one disk;
  `fix -f photos/` limits it to matching
  files. On a big array, scoping the fix
  saves hours.

## When `fix` says no

If the damage exceeds your parity — two
disks dead on single parity, or a dead
disk plus corruption elsewhere — the
affected files come back as:

```
unrecoverable movies/birthday.mp4.bin
DANGER! Unrecoverable errors detected!
```

and `fix` removes the hopeless remnants
rather than leaving corrupt files
pretending to be whole. That arithmetic is
exactly what chapter 8's second parity
changes: with two levels in place, the
same double failure — corruption plus a
disk loss — recovered in full. Files on the *surviving* disks are
never touched: you lose only what the
failed disks held, which is the sharpest
contrast with classic RAID, where a
failed rebuild can take the array.

## The checklist to keep

1. Stop writing to the array if you can.
2. `check` to learn the extent.
3. `fix` (scoped with `-d` or `-f` if the
   damage is local).
4. Verify with checksums, a media player,
   your own eyes — not with relief.
5. **Then** investigate the hardware. A
   disk that corrupted once will do it
   again; recovery is not repair.

## The one-sentence version

`fix` rebuilds corrupted, deleted and
whole-disk losses from parity and the
content file — provided the loss is within
your parity count and no sync has since
made the loss the truth.
