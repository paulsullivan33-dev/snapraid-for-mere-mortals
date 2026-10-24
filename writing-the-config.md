# Writing the Config

Everything SnapRAID knows about your array
comes from one plain-text file. This
chapter writes the lab's config line by
line; the same lines, with real mount
points, are a working config for a real
array. The full file is also in
`examples/snapraid.conf`.

## The lab's config

```
parity /home/hatch/workspace/snapraid-lab-work/parity/snapraid.parity
content /home/hatch/workspace/snapraid-lab-work/disk1/snapraid.content
content /home/hatch/workspace/snapraid-lab-work/disk2/snapraid.content
data d1 /home/hatch/workspace/snapraid-lab-work/disk1/
data d2 /home/hatch/workspace/snapraid-lab-work/disk2/
exclude /snapraid.content
exclude /snapraid.parity
```

On a real machine the paths would be
mount points — `/mnt/disk1/`, `/mnt/disk2/`,
`/mnt/parity/` — and the shape is identical.
Take the lines in turn.

## `parity` — where the recovery information lives

```
parity /mnt/parity/snapraid.parity
```

One line names the parity file. The parity
disk must have room for as much parity as
the **largest** data disk holds data —
parity grows block for block with the
fullest data disk, not with the total.
SnapRAID extends the file as the array
grows; a parity disk that fills up stops a
sync, it does not corrupt anything.

## `content` — the index, in several copies

```
content /mnt/disk1/snapraid.content
content /mnt/disk2/snapraid.content
```

One line per copy. The rules SnapRAID
enforces, and the lab met immediately:

- at least **two** copies;
- on **different disks** from each other;
- a copy may live on a data disk (normal)
  but never only on the parity disk's
  device.

Transcript t12 shows `status` refusing a
config whose content copies share a disk:
"You must have at least 2 'content' files
in different disks." Put a copy on every
data disk. If a disk dies, its copy dies
with it and the survivors carry on —
chapter 7's whole-disk recovery ran on
exactly that.

## `data` — the disks being protected

```
data d1 /mnt/disk1/
data d2 /mnt/disk2/
```

Each data disk gets a short **name**
(`d1`, `d2`, or anything you like — names
appear in reports and in `fix -d` later)
and its root path, with a trailing slash.
Disks can be different sizes, different
filesystems, already full of files. Order
matters only for readability.

## `exclude` — what to leave out

```
exclude /snapraid.content
exclude /snapraid.parity
```

Patterns are relative to the data disk's
root. Keep SnapRAID's own files out of the
array, and exclude the things parity
cannot help and only churns on: caches,
temporary downloads, a torrent client's
scratch folder. Every excluded file is a
file with no protection — exclude by
temperament, not by hope. The manual lists
further options (`include`, per-disk
excludes, `block_size`, `autosave` for
very large arrays); defaults serve until
you have a reason.

## The rules the program enforces

SnapRAID checks the config against the
hardware before it does anything, and
refuses — loudly, with `DANGER!` — when
the array cannot protect what it claims
to. The lab met three of these guards:

| Guard | Message (abridged) | Transcript |
|---|---|---|
| Two members on one device | "Disks '...' and '...' are on the same device." | t12 |
| Content copies not spread out | "You must have at least 2 'content' files in different disks." | t12 |
| A parity file smaller than the array expects | "One or more the parity files are smaller than expected!" | t11 |

Treat these refusals as the program doing
its job. In particular the second and
third are what stand between you and
syncing happily over an unmounted disk —
the classic way to destroy an array with
its own protection tool. Chapter 14 works
through each.

Check: a config is accepted when any
read-only command runs clean against it:

```
$ snapraid -c snapraid.conf status
SnapRAID status report:
...
No error detected.
```

## The one-sentence version

Name one parity file, spread content
copies across the data disks, list the
data disks with short names, exclude
SnapRAID's own files — and let the
program's refusals teach you the rest.
