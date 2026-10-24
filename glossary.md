# Glossary

- **Array** — the set of data disks and
  parity disks one config protects
  together.
- **Block** — the fixed-size chunk files
  are divided into for parity and hashing;
  256 KiB by default.
- **Check** — the command that re-reads
  and re-hashes the whole array now,
  verifying data and parity.
- **Content file** — SnapRAID's index:
  every file's name, size, timestamp and
  block hashes. Kept in several copies on
  different disks. Lose them all and the
  parity cannot be used.
- **Diff** — the command listing what a
  sync would change: added, removed,
  updated, moved. The pre-flight check.
- **Fix** — the command that rebuilds
  corrupted, deleted or lost files from
  parity and the content file.
- **Hash** — a short fingerprint of a
  block's contents; how corruption is
  *detected*, as opposed to repaired.
- **Parity** — recovery information
  computed from the data blocks, from
  which any one missing block in a stripe
  can be rebuilt. Also the disk, or the
  file, holding it.
- **Parity level** — one independent
  parity computation. `parity`, `2-parity`
  up to `6-parity`; each level survives
  one more simultaneous failure.
- **Pool** — SnapRAID's built-in merged
  view of the data disks, made of
  symlinks; a lighter alternative to a
  merging filesystem.
- **Scrub** — hash verification of a
  rotating slice of the array, remembering
  each block's last verification date.
- **Silent corruption (bit rot)** — data
  changed on disk without any error being
  reported; only a hash comparison notices.
- **SMART** — disks' built-in
  self-monitoring data; SnapRAID's `smart`
  command reads it to estimate disk
  health.
- **Stripe** — the set of blocks at the
  same position across all data disks,
  plus their parity block(s).
- **Sync** — the command that brings
  parity and the content file up to date
  with the disks as they are now — and,
  in doing so, makes the current state
  the protected one, deletions included.

## Sources

Checked against, in this guide's lab:

- The SnapRAID manual, on the project
  site: https://www.snapraid.it/manual
- The project and its releases:
  https://github.com/amadvance/snapraid
- The community all-in-one script:
  https://github.com/auanasgheps/snapraid-aio-script
- The MergerFS project:
  https://github.com/trapexit/mergerfs
