# Bash State

A Bash library to save and restore as much local Bash state as feasible, to
enable quickly starting interactive shells.

## Demo

To save state of current shell, run the following at the Bash command prompt:

```sh
source bash-dump-state >/tmp/bash-state
```

To restore state into another shell, run the following:

```sh
source /tmp/bash-state
```

## Installation

In your `~/.bashrc`, put something like this at the beginning:

```sh
# If not running interactively, don't do anything
[ -z "${PS1:-}" ] && return

BASH_LOAD_STATE=${BASH_LOAD_STATE:-1}
BASH_STATE_FILE=${BASH_STATE_FILE:-"/tmp/bash-load-state-${USER}-${EUID}"}
if [[ "${BASH_LOAD_STATE:-0}" -ne 0 ]] && [[ -r "${BASH_STATE_FILE}" ]]; then
    if [[ -O "${BASH_STATE_FILE}" ]]; then
        echo "${HOME}/.bashrc: loading from ${BASH_STATE_FILE}. If you need to reinitialize that file, run \`BASH_LOAD_STATE=0 bash -l\`" 1>&2
        # shellcheck disable=SC1090
        source "$BASH_STATE_FILE"
        return
    else
        echo "${HOME}/.bashrc: ${BASH_STATE_FILE} exists but is not owned by you. Will try to write to it at the end of this script, but will probably fail. You should investigate." 1>&2
    fi
fi
echo "${HOME}/.bashrc: loading full Bashrc. Will run \`source ~/bin/bash-dump-state >\"${BASH_STATE_FILE}\"\` afterward." 1>&2
```

And something like this at the end:

```sh
echo "${HOME}/.bashrc: Running \`source ~/bin/bash-dump-state >\"${BASH_STATE_FILE}\"\` to save state." 1>&2
mkdir -p "${BASH_STATE_FILE%/*}"
# shellcheck disable=SC1090,SC1091
source ~/bin/bash-dump-state >"${BASH_STATE_FILE}"
```

Now all Bash shell state will be saved into `${BASH_STATE_FILE}` if it doesn't
exist. This file will be used to quickly load the state upon subsequent
shells, unless the environment variable `BASH_LOAD_STATE` is unset, empty, or 0.

## TODO

- [ ] Convert to `bash_state_save` and `bash_state_load` shell functions.
- [ ] Lock in `bash_state_save` to avoid having multiple shells stomp on each
      other during startup (`flock` on Linux and where present, `shlock` as
      fallback).
- [ ] Make compatible with Bash 3.x if possible (right now requires at least
      Bash 4.4 for `${x@Q}` syntax).
- [ ] Convert beginning and end blocks in `~/.bashrc` to shell functions.
