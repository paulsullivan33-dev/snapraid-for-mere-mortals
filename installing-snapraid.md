# Installing SnapRAID

SnapRAID is a single self-contained
program. There is nothing to configure at
install time and nothing runs afterwards
until you invoke it. This chapter covers
the package, the source build the lab
performed, and Windows in brief.

## The declared deviation, up front

The lab's array is made of **directories
on one filesystem**, standing in for
disks. SnapRAID normally refuses this: it
checks the device behind each path and
stops if two members of the array turn out
to live on the same device — quite
rightly, since parity on the same physical
disk as the data protects against nothing.
Every lab command after chapter 4
therefore carries SnapRAID's own test
option, `--test-skip-device`, which turns
that refusal into a warning and lets the
workflow run. Transcript t12 shows the
refusal happening without the flag, so you
can see both behaviours.

Everything else in the lab is the genuine
workflow: real parity computed over real
files, real corruption repaired from it.
What the deviation cannot prove is
anything about physical devices — SMART
data, spin states, temperatures — and
those are labelled documented-only
wherever they appear.

## Option 1: your distribution's package

On Debian and Ubuntu, SnapRAID is in the
**universe** repository:

```
sudo apt install snapraid
```

Check: the version prints its name and
author:

```
$ snapraid --version
SnapRAID CLI v15.0 by Andrea Mazzoleni, https://www.snapraid.it
```

(The lab's binary says 15.0 because it was
built from the 15.0 source. A packaged
install may be a release or two behind the
project's latest — for SnapRAID that is
normally fine, and chapter 10 covers
upgrading deliberately rather than
accidentally.)

## Option 2: build from source (what the lab did)

The release tarball is on the project's
GitHub releases page. The build is the
classic three steps, needing only a C
compiler and `make`:

```
tar xzf snapraid-15.0.tar.gz
cd snapraid-15.0
./configure
make
sudo make install
```

The lab ran `./configure` and `make` in
this sandbox and used the resulting binary
in place, without the `make install`
step; the project's own `make check`
regression suite exists and takes a few
minutes if you want the extra assurance
before installing.

## Windows

SnapRAID ships Windows binaries (32- and
64-bit) on the same releases page; the
program is the same, the config uses drive
letters (`data d1 E:\`), and the commands
in this guide translate directly. The lab
ran on Linux only, so Windows specifics —
service scheduling, path quirks — are
documented-only in this guide and the
project's manual (which has a Windows
section) is the source to trust.

## Where things go

Two placements to decide now:

- **The binary**: wherever your package
  manager or `make install` puts it,
  conventionally `/usr/bin` or
  `/usr/local/bin`.
- **The config**: SnapRAID looks for
  `snapraid.conf` in the current directory
  and in system locations (`/etc/snapraid.conf`
  on Linux). Chapter 4 writes one and every
  later command names it explicitly with
  `-c`, which is the habit to keep: the
  config in use is never a guess.

## The one-sentence version

Install the package if your distribution
has one; build the tarball if it doesn't;
either way you end with one binary, no
daemon, and a version command that proves
it works.
