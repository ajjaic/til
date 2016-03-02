# Fixing Wonky Colors

If a vim session inside a tmux window shows wonky colors then it might help to
start tmux with `tmux -2` instead. The `-2` option tells tmux to assume that
the terminal supports 256 colors

# Adjusting Window Pane Size

In tmux, the size of window panes can be adjusted incrementally with the
`resize-pane` command. For instance, to resize a pane in any direction
(left, down, up, and right), use one of the following commands

```
resize-pane -L 10
resize-pane -D 10
resize-pane -U 10
resize-pane -R 10
```

This will adjust the pane by *10* units in the corresponding direction. Of
course, other values can be used for more significant size adjustments.

# Create A Named tmux Session

When creating a new tmux session

```bash
$ tmux new
```

a default name of `0` will be given to the session.

If you'd like to give your session a name with a bit more meaning, use the
`-s` flag

```bash
$ tmux new -s burrito
```

[source](https://robots.thoughtbot.com/a-tmux-crash-course)

# Cycle Through Layouts

Arranging a series of split windows in tmux can take some time. Once those
splits windows are arranged, it is difficult to set them up in a new way.
There is a way of cycling through layouts that might be able to help though.
Hit `<prefix><space>` over and over to cycle through the layouts until you
find the arrangement that you want.

[source](http://superuser.com/questions/493048/how-to-convert-2-horizontal-panes-to-vertical-panes-in-tmux)

# Enabling Vi Mode

If you'd like some vim-like bindings in tmux, you can turn on vi mode. To do
so, add the following line to your `.tmux.conf` file:

```
setw -g mode-keys vi
```

With this added and tmux re-sourced, you'll now have a variety of vi-like
bindings. For instance, when in tmux's copy mode, you can now use `v` to
begin making a visual selection and `y` to yank that selection into tmux's
copy buffer.

# Jumping Between Sessions

If you are using tmux to manage multiple projects by putting each project in
a separate session, you may find yourself moving between sessions
frequently. Detaching and reattaching can and will get tedious. There are
better ways. tmux provides the `<prefix>)` and `<prefix>(` bindings as a way
of jumping to the next and previous session, respectively. That should
reduce some friction.

# List All Key Bindings

There are a couple ways to list all the tmux key bindings. If you are not
currently in a tmux session, you can still access the list from the terminal
with

```bash
$ tmux list-keys
```

If you are currently in a tmux session, then you can take advantage of the
tmux environment by either using

```
<prefix>:list-keys
```

or

```
<prefix>?
```

Any of these will bring up a list of all key bindings available to you
within a tmux session on your machine.

# List Sessions

Not sure if `tmux` is running or, if it is, which sessions are available?
You can list all the currently running sessions right from the command-line.

```bash
$ tmux ls
```

This command is shorthand for:

```bash
$ tmux list-sessions
```

# Open New Window With A Specific Directory

When you initially start a tmux session, the default directory is based off
of whatever the current working directory was. Any subsequent windows opened
within that tmux session will be opened with that as the base directory.

To open a window with a different default directory, use the `-c` flag with
the `new-window` command. For example, hit `<prefix>:` and then

```
:new-window -c ~/
```

to open a new window with your home directory.

[source](http://unix.stackexchange.com/questions/12032/create-new-window-with-current-directory-in-tmux)

# Organizing Windows

If you use a number of tmux windows as part of your daily workflow, you may find that they get to be a bit of a mess from time to time. There are gaps in the numbering and they aren't laid out in the order you'd prefer. The `movew` command makes it easy to rearrange these windows.

If you have a window indexed at 2 and you want it to be the 4th window, then you can:

```
:movew -s 2 -t 4
```

If you have a gap such that the 4th and 5th windows are numbered 4 and 7, then you can focus the 7 window and simply invoke:

```
:movew
```

And that window will be reinserted at the next available slot, in this case, window 5.

# Paging Up And Down

When in _copy mode_ (`<prefix>[`), you can move the cursor around like you
would in vim with the directional keys (`hjkl`). This works fine until you
want to move up or down through pages and pages of text, such as when
navigating to the top of a long stack trace. One way to get where you need
to be more quickly is by paging up and down.

Hit `CTRL-u` to page up and `CTRL-d` to page down.

# Pane Killer

The current pane can be _killed_ (closed) using the following key binding:

```
<prefix>x
```

You will be prompted to confirm with either `y` or `n`.

If there is only one pane in the current window, then the window will be
_killed_ along with the pane.

# Reclaiming The Entire Window

If you have attached to a tmux session whose dimensions are being
constrained by another connection, you may find an L-shaped portion of your
window filled with dots. tmux defers to the session with smaller dimensions.
The easiest way to reclaim the entire window for your session is to attach
to the session while forcing all other sessions to detach. The `-d` flag will
help with that.

```bash
$ tmux attach-session -d -t my-session
```

By detaching all other sessions, you are ensuring that your machine's
dimensions are the ones that tmux uses when drawing the window. This is a
great quick fix if you're working on your own, but probably not what you
want to do if you are in a pair programming situation.

[source](http://stackoverflow.com/questions/7814612/is-there-any-way-to-redraw-tmux-window-when-switching-smaller-monitor-to-bigger)

# Rename The Current Session

If you've created an unnamed tmux session or you no longer like the original
name, you can open a prompt to change it by hitting

```
<prefix>$
```

Replace the existing name with the desired name and hit enter.

h/t Dorian Karter

# Swap Split Panes

Use `<prefix>}` (or `<prefix>{`) to swap two vertically or horizontally
split tmux panes.

# tmux in your tmux

If you are running tmux locally and you shell into another machine to
access tmux remotely, you will suddenly find yourself in tmux inception.
You will have a tmux instance running within your local tmux instance. If
you have the same prefix key set for both, then you may be wondering how
you can send a tmux command to the *inner* tmux instance.

If you press your prefix twice (e.g. `<C-a> <C-a>`), then the second prefix
will be sent to the inner tmux instance which will then be listening for
the rest of your command. So, to open a new window within the inner tmux
instance, you can hit `<C-a> <C-a> c`.
