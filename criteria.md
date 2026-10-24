# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.


## Verification notes for this guide

The 9-point standard above governed this guide.
How it was met, specifically:

**Installed and run for real:** SnapRAID 15.0
(the project's current release on build day),
compiled from the release tarball in this
sandbox (`./configure`, `make`) and used in
place. A two-data-disk, one-parity array was
built, then a second parity added. The full
transcripts are in `snapraid-lab/transcripts/`
(t00–t12) and are summarised in
`examples/lab-notes.txt`.

**One deviation, declared:** the array's disks
are directories on a single filesystem, so every
lab command after the config chapter carries
SnapRAID's own test option `--test-skip-device`.
Without it SnapRAID refuses the array for being
on one device — transcript t12 captures that
refusal for both `sync` and `status`. The parity
computation, hashing, corruption detection, and
recoveries are the program's real code paths and
were verified against SHA-256 checksums recorded
before SnapRAID ran; what the deviation cannot
prove is anything about physical devices.

**Ran and verified:** build from source; version;
config with parity, two content copies, two data
disks, excludes; first `sync` (parity file
13,631,488 bytes; "Everything OK"); `status`;
`diff` (exit 2 with differences, 0 without);
second sync; `scrub -p 100 -o 0`; `check` clean;
a file corrupted in place with `dd` — `check`
fails (exit 1, "DANGER! Unexpected silent data
errors!") — `fix` recovers it, checksum OK; a
file deleted, `fix` recovers it; a file deleted
**then synced over**, `fix` finds nothing —
deletion made permanent, shown on purpose; a
whole data disk emptied, `fix` rebuilds every
file from parity (its content copy died with it
and a survivor's copy was used); second parity
added to the running array — plain sync refuses
with instructions, `--force-full sync` builds it —
then a corruption and a disk loss at the same
time, `fix` recovers everything, all checksums
OK; and the same-device and content-spread
refusals captured in t12.

**Documented only, from the project's manual:**
hardware-facing features the directory lab
cannot reach — `smart` health reports, drive
probe/spin behaviour, temperature handling;
the `pool` symlink view; split parity; 3-parity
and above (the mechanism is the same; only 1 and
2 levels were run); interrupted-sync recovery;
Windows builds (the program and config translate;
nothing Windows-specific was run); scheduling
under cron/systemd is given as a pattern, not a
lab result, as is the community all-in-one
script's behaviour and the MergerFS pairing in
the recipes chapter. Upgrading and array-change
procedures follow the fix/sync paths proven in
the lab but were not separately rehearsed.
Chapter 12 compares other systems from their
own documentation; none was run.

**Fixtures:** all invented and generic — photos,
a family video, films, music files, a shopping
list, a seed catalogue — filled with random bytes
and checksummed. No real person's data, machine
names, or organisations appear in the lab.

**Numbers:** sizes and exit codes quoted are the
lab's own outputs, kept in the transcripts. No
speed, rebuild-time, or reliability figures are
claimed — the array is megabytes, and timings at
that scale would be theatre. Hardware sizing in
the text (parity matches the largest data disk)
is the program's rule, not a benchmark.
