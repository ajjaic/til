# Directional Commands

You can move the cursor without arrow keys. Here is the keyboard equivalent for each.

* Up ('previous'): `CTRL + P`
* Down ('next'): `CTRL + N`
* Left ('back'): `CTRL + B`
* Right ('forward'): `CTRL + F`

Mapping `caps lock` to `CTRL` makes these combinations very accessible.
# Finding Getters

After writing the other day about why you [might not want to use simple getters](http://til.hashrocket.com/posts/7d6b8eb8d3-accessor-performance-gap), I decided that I wanted to eliminate all such methods from a project.

Here is the regex I wrote to isolate the pattern:

```
ag 'def (\w+);?\s+@\1;?\s+end'
```

The semicolon catches one-line getter methods as well as the more common three-line.
# MD5 File Signatures

The command `md5 <file>` generates a unique 32-digit hexadecimal number. This can serve as a signature for a file in its particular state, letting you know when it has changed.

Example usage:

```shell
$ touch test.txt
$ md5 test.txt
  MD5 (test.txt) = d41d8cd98f00b204e9800998ecf8427e
$ echo 'some content' > test.txt
$ md5 test.txt
  MD5 (test.txt) = eb9c2bf0eb63f3a7bc0ea37ef18aeba5
```
# Printing with lpr

Recently while trying to fix a printer I used `lpr` a bunch of times. It's not exactly new to me, but never fails to surprise people when I use it.

`lpr` submits files for printing to your default printer in OSX.

Print a file:

```
lpr README.md
```

Print the current file in your Vim session, with a *cool job name*:

```
:! lpr -T 'cool job name' %
```

Print two copies to a specific printer:

```
lpr -# 2 -P specific_printer README.md
```

This is an invaluable command-line trick.
# Reverse a String

Reverse a string with the `rev` command.

```shell
$ echo 'test' | rev
tset
```

It also works with files.

```shell
$ cat Procfile | rev
br.amup/gifnoc C- amup cexe eldnub :bew
```
# Run Previous Command

Previously run commands can be viewed with the `history` command.

```shell
$ history
10048  git checkout master
10049  gpr
10050  rake
```

With this list, you can rerun any command using `!n`:

```shell
$ !10048
Already on 'master'
```

The command `!!` prints the last command you ran, then runs it. Here is an example:

```shell
$ ls
LICENSE.md README.md  bash       cucumber   rails
$ !!
ls
LICENSE.md README.md  bash       cucumber   rails
```

Replace the second `!` with the first few letters of a command you have previously run, and bash will search for, print, and run the most recent instance.

```shell
$ !rsp
rspec spec/models/user.rb
...
```
# Screen Flashing in OSX

I like a quiet computer. But I also like tab-completing commands in the terminal, which produces a necessary alert from time to time.

A solution I found today is to use OSX Accessibility features to add a screen flash on alerts, then disable the sounds.

System Preferences > Accessibility > Hearing menu, choose 'Audio' > Check 'Flash the screen when an alert occurs'
# Send Processes to the Background

Processes on any POSIX-compliant computer can be sent to the background with `CTRL-Z` (`<prefix> + Z` for the tmux-ers) and returned to the foreground with `fg`.

Here is an example:

```shell
user@computer:~% ping www.google.com
PING www.google.com (74.125.228.212): 56 data bytes
64 bytes from 74.125.228.212: icmp_seq=0 ttl=52 time=41.574 ms
64 bytes from 74.125.228.212: icmp_seq=1 ttl=52 time=42.836 ms
64 bytes from 74.125.228.212: icmp_seq=2 ttl=52 time=53.527 ms
^Z
zsh: suspended  ping www.google.com
user@computer:~% fg
[1]  + continued  ping www.google.com
64 bytes from 74.125.228.212: icmp_seq=3 ttl=52 time=42.433 ms
64 bytes from 74.125.228.212: icmp_seq=4 ttl=52 time=42.401 ms
64 bytes from 74.125.228.212: icmp_seq=5 ttl=52 time=42.837 ms
64 bytes from 74.125.228.212: icmp_seq=6 ttl=52 time=44.203 ms
^C
--- www.google.com ping statistics ---
7 packets transmitted, 7 packets received, 0.0% packet loss
```
Adding `| pbcopy` to the end of any command will send the standard output to your clipboard.

`ls -al | pbcopy` inside a rails project directory allowed me to paste this:

```
Gemfile
Gemfile.lock
README.md
Rakefile
app
bin
config
config.ru
db
features
lib
log
public
script
spec
test
tmp
vendor
```
# Watch That Program

Have you ever been working in the terminal and found yourself repeating the same command many times? Delegate that work to the computer.

`watch` comes with Linux and can be installed on OSX via homebrew. It executes a program periodically, defaulting to every two seconds.

We used it today while writing a database backup script. Instead of checking our dump directory every time a cron job executed, we ran `watch ls`, and watched the script succeed or fail with live updates.

`man watch` for more information.
