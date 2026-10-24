# The Moving Parts

SnapRAID has no database server, no
daemon, and no kernel module. It is one
program, one config file, and two kinds of
file it maintains on your disks. This
chapter names them all, because nearly
every confusion later is really a question
about which of these holds what.

## The pieces

| Piece | Where it lives | What it holds |
|---|---|---|
| `snapraid` | Your system | The program itself |
| `snapraid.conf` | You choose (often `/etc`) | Which disks, which parity, what to exclude |
| **Parity file** | The parity disk | The recovery information itself |
| **Content file** | On the data disks (several copies) | The index: every file, its size, timestamp and hashes |
| Your files | The data disks | Untouched, in their usual places |

## Blocks and hashes

SnapRAID does not work file by file when
it computes parity. It chops every file
into **blocks** — 256 KiB each by default —
and lines the blocks up across the data
disks: block 1 of disk 1, block 1 of disk
2, and so on form a *stripe*, and the
parity file holds one block per stripe
that can rebuild any one missing block in
that stripe. (With a second parity it
holds a second, different computation over
the same stripe, which is what makes two
failures survivable.)

For every block, the content file stores a
strong **hash** — a short fingerprint of
the block's contents. The hash is what
turns parity from mere rebuild machinery
into a corruption detector: at any time,
SnapRAID can re-read a block, re-hash it,
and compare. If the hash differs, the data
on disk is not the data that was synced,
whatever the filesystem believes. That
comparison is what `check` and `scrub`
do all day (chapter 6).

## The content file is the index of everything

The content file deserves its own warning
box. It records, for every file in the
array: the name, the size, the timestamp,
and the hash of every block. Without it,
the parity file is a pile of numbers with
no map — SnapRAID cannot tell which block
belongs to which file, and recovery is
effectively impossible.

So SnapRAID keeps **several copies** of
the content file, on different disks, and
chapter 4's config puts one on each data
disk. Losing one copy is a non-event: the
next command uses another and rewrites the
missing one (the lab's transcript t11
shows exactly this message when a disk
vanished with its copy). Losing *every*
copy is the real disaster, worse in some
ways than losing a data disk — guard the
content file like the data, and let a
backup job pick up a copy of it too.

Check: the lab's content files after the
first sync were small enough to see whole:

```
$ ls -la disk1/snapraid.content disk2/snapraid.content
-rw------- 1 root nogroup 1931 ... disk1/snapraid.content
-rw------- 1 root nogroup 1931 ... disk2/snapraid.content
```

Small array, small index. It grows with
the number of files, not their size.

## What a sync actually changes

When you run `sync`, SnapRAID:

1. reads the content file — its picture of
   the array at the last sync;
2. scans the data disks for files that are
   new, changed, moved or gone;
3. reads only the changed blocks, updates
   the parity for their stripes;
4. writes the new content file and verifies
   it.

Files are recognised by name, size and
timestamp (down to fractions of a second).
A file whose contents changed but whose
size and timestamp somehow did not — the
signature of silent corruption — is
invisible to `sync`. It is *not* invisible
to `check` and `scrub`, because those
compare hashes. That asymmetry is the
reason both commands exist, and chapter 6
returns to it.

## The one-sentence version

Two files do all the work: the parity
file, which can rebuild a missing block,
and the content file, which remembers
which block is which and what it should
hash to — protect copies of the second
like you protect the data itself.
