# Troubleshooting

The failures below are the ones this
guide's lab produced on purpose or by
accident, plus the standard field cases
from the project's documentation. Each is
symptom, cause, action.

## "Disks '...' and '...' are on the same device"

*Transcript t12.* Two members of the array
resolve to the same physical device — in
the lab, because directories shared one
filesystem; in the field, because a path
is wrong (a typo pointing two `data`
lines at one mount) or a disk failed to
mount and its path fell back to the root
filesystem. **Action:** check mounts
(`lsblk`, `df`), fix the path, never work
around it by hunting for the test flag.
The refusal is the protection working.

## "You must have at least 2 'content' files in different disks"

*Transcript t12.* Fewer than two content
copies, or all copies on one device.
**Action:** add `content` lines spread
across the data disks (chapter 4). If you
meet this after a disk died, it is
telling you the surviving copies are all
on one disk — proceed, then restore the
spread when the disk is replaced.

## "The parity has only 0 blocks" / "parity files are smaller than expected"

*Transcript t11.* The parity file is
shorter than the content file says it
should be. Two causes, and SnapRAID
cannot tell them apart — which is the
point of refusing:

- The parity disk **is not mounted** (the
  file found is a stub on the root
  filesystem). Mount it, and the numbers
  agree.
- You are **adding a parity level**
  (chapter 8). Follow the printed
  instructions: initialise with
  `snapraid -d <level> fix` then sync, or
  rebuild with `--force-full sync`.

Never force your way past this one
without knowing which cause you have.

## "DANGER! Unexpected silent data errors!"

A `check` or `scrub` found blocks whose
hashes disagree — corruption, found.
**Action:** chapter 7's checklist.
`fix` repairs from parity; then find out
*why* (SMART data, cables, a disk at the
end of its life) before the next block
goes. A growing error count across runs
is a disk announcing its retirement.

## `fix` reports a file "unrecoverable"

The damage exceeds your parity: typically
a dead disk plus damage on a survivor, or
a file deleted *and synced over* before
you tried to recover it (that one is not
damage but arithmetic — transcript t09).
**Action:** restore that file from
backup; the rest of the array is
unaffected. If files are unrecoverable
often, the answer is another parity level
or better hardware hygiene, not hope.

## Sync is suddenly enormous, or everything is "updated"

Usually timestamps: files whose timestamps
lost sub-second precision in a copy (some
tools and filesystems round them) stop
matching the content file and look
changed. The `touch` command re-stamps
files SnapRAID cannot verify for this
reason; `status` reports how many files
have zero sub-second timestamps — the lab
array's reports proudly showed none.
**Action:** run `touch`, let the next
sync settle, and prefer copy tools that
preserve timestamps (`rsync -a`).

## A content file is missing

"No content file found. Assuming empty."
— SnapRAID opened with one copy and
adapted (transcript t11 shows it falling
back). **Action:** after any disk swap,
run `status` and confirm the content
copies are back to one per disk — any
command rewrites a missing copy from a
surviving one. If *all* copies are gone,
stop: do not sync (an "empty" array
would overwrite the parity picture), and
restore a content copy from backup. This
is chapter 2's warning with its sleeves
rolled up.

## The one-sentence version

SnapRAID's errors are specific and mean
what they say — a device that is not what
the config claims, an index not spread
out, a parity shorter than the truth, or
damage past your parity count — so read
the message, fix the named thing, and
never argue with a `DANGER!`.
