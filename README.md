# rsync-mover

Menu-driven push/pull between the current directory and a remote host.

`mover` for the menu, `mover -h` for the CLI.

## Usage

First run writes a blank `mover.conf` beside the script and stops. Fill it in:

```sh
HOST=user@host          # required
KEY=~/.ssh/id_ed25519   # empty = let ssh choose
REMOTE_ROOT=src         # empty = remote home
EXCLUDES="node_modules .venv"   # empty = none
DELETE_MODE=ask         # ask | yes | no
DRY_RUN_MODE=ask        # ask | yes | no
GITIGNORE_MODE=ask      # ask | yes | no
```

One file per host: `nas.conf` → `-H nas` for one run, `MOVER_PROFILE=nas` for
a shell, or `p` in the menu, which is remembered for next time (in
`$XDG_STATE_HOME/mover/profile`).

`MOVER_CONFIG_DIR` moves the profiles elsewhere.

`mover check` verifies the connection and both rsync versions.
`source <(mover completion)` for completion; remote names come from a cache
that `mover ls` seeds.

At any path prompt, Ctrl-D or clearing the line (Ctrl-U, Enter) returns to the
menu. 

Requires bash 4.4+, GNU coreutils, and rsync 3.2.3+ at both ends.

## Path rules

Local root is always `$PWD`. Remote paths are anchored under `REMOTE_ROOT`
unless absolute. Trailing slashes behave as rsync's, except that mover adds
one when it derives the destination itself, so `push ~/src/proj` can't nest
`proj` inside `proj`.

## Notes

- Excluded files are also protected from deletion, so `-d` won't clear
  `node_modules` off the remote.
- `-d` caps deletions at 1000 per run (`-m NUM`, `-m 0` to warn without
  deleting, `-m none` for no cap). Hitting the cap deletes up to the cap and
  skips the rest, exiting 25.
- The delete confirmation only appears on a TTY; scripted runs with `-d`
  proceed unprompted.
- `--mkpath` is always on; needs rsync 3.2.3+ at both ends.
- `RemoteCommand=none` clears a `RemoteCommand` in *your* `~/.ssh/config`. It
  does not override a forced `command=` in the remote's `authorized_keys` —
  use `rrsync` for that.
- `BatchMode=yes` also disables host-key prompts, so a first connection to an
  unknown host fails. Add `StrictHostKeyChecking=accept-new` to
  `BASE_SSH_OPTS` if you'd rather it connect.
- Connections are multiplexed for 60s, keyed per profile.
- `-a` doesn't cover hard links, ACLs or xattrs.
