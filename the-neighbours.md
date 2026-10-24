# The Neighbours

SnapRAID is one answer to "a disk died"
among several, and it is not always the
right one. This chapter lines up the
alternatives and names what each trades.
No neighbour was run in this guide's lab;
these are design comparisons, with each
project's own documentation as the place
to verify specifics.

| Approach | Protects against | Costs | Watch out for |
|---|---|---|---|
| **SnapRAID** | Disk death, silent corruption, recent deletions | One disk per parity level; sync discipline | Nothing between syncs |
| **Classic RAID 5/6** (mdadm, hardware) | Disk death, in real time | Matched-ish disks, all spinning | Rebuild failures; whole array at risk; not corruption-proof |
| **ZFS** (RAIDZ, mirrors) | Disk death, corruption, with snapshots for history | RAM and matched vdev planning | Growing an array is rigid; snapshots are not backups |
| **Unraid** | Disk death, one or two parity disks | A licence; its own OS | Real-time parity writes are slow; it's a platform choice, not a tool |
| **Btrfs** | Corruption detection, snapshots, RAID profiles | Less parity tooling than SnapRAID | Its RAID 5/6 has a long-documented write-hole history |
| **Plain backup** (rsync, restic, a second copy) | Everything, including fire and deletion-with-history | A full second copy's space | No help the day the only disk dies mid-week, if the backup is weekly |

## The distinctions that matter

**Real-time vs snapshot.** RAID, ZFS and
Unraid protect every write as it happens;
SnapRAID protects at sync time. If your
files change constantly and every version
matters, real-time wins — and so does a
real database backup, because none of
these is one.

**Corruption.** Classic RAID happily
mirrors corrupted data to every disk; it
has no hashes. SnapRAID, ZFS and Btrfs
hash blocks and can say *which* copy is
right. On ageing disks this difference is
the whole game.

**History.** Only snapshots and backups
have a past. Parity of any flavour
protects the present arrangement of
bytes. Delete-and-sync, ransomware, a
house fire — parity shrugs; a backup (or
a snapshot you kept) answers.

**Exit cost.** SnapRAID and plain backup
leave your disks as ordinary disks
throughout. RAID and ZFS own the layout:
leaving means copying everything off.

## The usual honest answer

For the home case this guide targets, the
common settled answer is **SnapRAID for
the shelf, plus a backup for the
irreplaceable** — parity absorbs the disk
failure, the backup absorbs the mistakes
and disasters. If you find yourself
wanting real-time protection, checksums,
snapshots *and* painless growth in one
system, that list is ZFS's home ground
and this guide will not pretend
otherwise.

## The one-sentence version

Parity answers "a disk died", snapshots
answer "last Tuesday", backups answer
"everything is gone" — pick the tools by
which question keeps you up at night.
