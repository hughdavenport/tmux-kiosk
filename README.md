# tmux-kiosk
Run a command in a tmux pane, but locked down like a kiosk

The use case is to run a .desktop application within a tmux shell, but not allow the user to "break" out into other tmux sessions.
Of course there are likely ways that the user could break out, but as a basic kiosk it's probably a good start.

This can allow users to copy data into the tmux paste buffer (and system clipboard if configured).

## Default key bindings
In kiosk mode, there is no `prefix`, you can't interact with the running command, and the only tmux key bindings are as follows:
- `?`: Display key binds
- `x`: Kill the current window and detach client
- `X`: Kill the current window but stay attached
- `r`: Restart current window with the same command
- `d`: Detach client
- `n`: Next window
- `p`: Previous window

No `prefix` is needed (and won't work). There is a patch implementing [a hook for command-error](https://github.com/tmux/tmux/pull/3973),
without which there are some cases that the `root` key table is selected (but without a `prefix`). If this happens, scrolling the
mouse up and down should return the user to the kiosk key table.

## How it works

Under the hood, this creates a new tmux `session` and a `keytable`. The key table defines a bunch of keys that force the client
to remain in that same table, along with the keys described above. After detaching, if it is the last client attached and all the
windows are dead, then everything is cleaned up.

## Debug mode

There is a debug mode, if the environment variable `TMUX_KIOSK_DEBUG` is non empty then the following extra keys will be defined:
- `C`: Customize mode (see `tmux` man page)
- `:`: tmux command prompt

This can allow power users to always be able to run tmux commands with `:` (no `prefix`). For example a user could run:
`set prefix C-b` then `switch-client -T root` to return to a mode more familiar. However note that the hooks will likely
return you to the kiosk mode. You will always have access to `:` in kiosk mode if the command was run with `TMUX_KIOSK_DEBUG`
set, on all clients of that session.

## Issues

Apologies if this causes any issues for users. If you do find an issue, please open a [github issue ](https://github.com/hughdavenport/tmux-kiosk/issues)
and consider making a pull request. The source is in bash script.
