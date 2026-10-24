# Your First Sync

This chapter runs the array for the first
time: the first `sync`, a look at
`status`, and the `diff` habit that should
precede every later sync. Everything here
was run in the lab; transcripts t02–t04
are the full output.

## Before: a "before" picture

The lab recorded checksums of every file
before touching SnapRAID (transcript t01).
Do the equivalent on a real array in
spirit: know roughly what is on your
disks and how full they are. The checksums
are what later chapters verify recoveries
against — recovery claims should always
end with a check, not a feeling.

## `sync` — bring the parity up to date

```
$ snapraid -c snapraid.conf sync
...
Everything OK
```

The first sync reads every file, computes
every block's hash, writes the full parity
file and the content files, then verifies
what it wrote. On the lab's 24 MB it took
seconds; on several terabytes of real
media it takes hours, once. Later syncs
only touch what changed.

Check: the parity file exists and is about
the size of the fullest data disk's data:

```
$ ls -la parity/ disk1/snapraid.content
-rw------- 1 ... 13631488 ... snapraid.parity
-rw------- 1 ...     1931 ... snapraid.content
```

(The lab's data disks held about 13.6 MB
between them; parity tracks the larger
disk's usage, block for block.)

## `status` — the array at a glance

```
$ snapraid -c snapraid.conf status
SnapRAID status report:

   Files Fragmented Excess  Wasted  Used    Free  Use Name
       3       0       0    -0.0       0     203   0% d1
       4       0       0     0.0       0     203   0% d2
```

Below the table, `status` draws the
scrub-age graph (chapter 6) and ends with
the lines worth reading every time:

```
No sync is in progress.
100% of the array is not scrubbed.
No error detected.
```

"No error detected" means no command has
*found* an error so far — it is a report
of the array's record, not a fresh proof.
Fresh proof is `check`'s job (chapter 6).

## `diff` — the pre-flight check

`diff` compares the disks against the
content file and lists what a sync *would*
do, without doing it:

```
$ snapraid -c snapraid.conf diff
update music/new-track.flac.bin

       6 equal
       0 added
       0 removed
       1 updated
       0 moved
       0 copied
       0 relocated
       0 restored
There are differences!
```

Exit code 0 means no differences; 2 means
differences were found (transcript t04).
Read the categories before every sync:

- **removed** is the dangerous one. A few
  removals you made on purpose are fine. A
  disk's worth of removals means a disk is
  missing, unmounted, or empty — **do not
  sync**; find out why first. Sync makes
  the current state the protected state,
  and chapter 7 shows what that does to a
  file deleted by accident.
- **added / updated** are the normal churn
  of a living array.
- **moved / copied** are recognised and
  cost no parity work.

## The rhythm to learn

```
snapraid -c snapraid.conf diff     # look first
snapraid -c snapraid.conf sync     # then commit
snapraid -c snapraid.conf status   # confirm
```

The lab's second sync (t04) shows the
steady state: one new file, a short sync,
"Everything OK". Chapter 9 turns this
rhythm into a schedule, with one
protection added: a limit on how many
deletions a scheduled sync may accept
before it refuses and asks a human.

## The one-sentence version

`sync` makes the parity match the disks as
they are right now, `diff` shows you what
that would mean before you commit to it,
and `status` is the report card you read
afterwards.
