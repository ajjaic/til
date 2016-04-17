# Mounting/Unmounting an NFS share on the client

To mount a NFS share on the client you can use
```
$ sudo mount <ip of NFS server>:<share path> <localpath to mount to>
```

To unmount the mounted filesystem,
```
$ sudo umount <localpath to mount to>
```

# Port forwarding and Port triggering
* Port forwarding:
  > New connections from the outside to a certain port or port range go to a
    designated LAN machine. The ports are determined by the kind of server you
    want to run, (e.g. 80 for a web server) and the IP is the private LAN IP
    of your web server.
* Port triggering:
  > new connections from the outside to a certain port go to whatever LAN
    machine made a certain outgoing connection (as defined by destination port).
    Example: You define port 25 as trigger and 113 as port. If any of your LAN
    machines creates a outgoing connection (=trigger) to port 25 (e.g. to send
    mail), all incoming connections to port 113 will temporarily go to that
    machine. After a timeout, new 113 connections will again be dropped.

# The Art of Grepping

* **Grep For Files Without A Match**
  > `$ grep -L "foobar" ./*` finds files that don't have the pattern "foobar"
    in the current directory. For more [info](http://stackoverflow.com/questions/1748129/using-grep-to-find-files-that-dont-contain-a-given-string-pattern)
* **Grep For Multiple Patterns**
  > `$ grep -e ruby -e clojure README.md` finds files in the current directory
    that contain both patterns "ruby" and "clojure"
* **Searching files recursively for a pattern**
  > `$ grep -r netlogo *` searches all files in the current folder for the
    pattern `netlogo` and then recursively descends into each directory to
    search for the pattern.

# Writing ISO/Image to USB

```
$ dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync
```

**of** and **if** stand for **output file** and **input file** respectively
The `of` should be the actual drive and as such has no partition number

# Show all open ports

To display the list of all open ports, enter:

```bash
$ lsof -i
```

# Check Ubuntu Version

Are you on Ubuntu? Want to know what version (release) of Ubuntu you are
using?

Run the following to find out:

```bash
$ lsb_release -r
```

# Configure Your Server Timezone

On Ubuntu and Debian, you can open an interactive prompt for configuring
your timezone with the following command:

```bash
$ dpkg-reconfigure tzdata
```

# Upgrading Ubuntu

I recently discovered that my Linode box was running a fairly old version of
Ubuntu. Because it is a remote box that I SSH into, there is no graphical
user interface. Upgrading to a newer release can be accomplished with the
following command line utility:

```
$ do-release-upgrade
```

It includes a series of prompts regarding choices about the upgrade and a
lot of waiting.

Adding the `-d` flag will upgrade to the latest development release.

# All The Environment Variables

If you want to see all the environment variables defined on your machine,
you can list them all out with `printenv`. If you are like me, you probably
have a ton of them. Pipe it through `less` to make it easier to navigate and
search through (i.e. `printenv | less`).

# Cat A File With Line Numbers
```
$ cat -n Gemfile
 1  source 'https://rubygems.org'
 2
 3
 4  # Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
 5  gem 'rails', '4.2.0'
 6  # Use postgresql as the database for Active Record
 7  gem 'pg'
```
[source]('https://rubygems.org')

# Change Default Shell For A User

You can change the default shell program for a particular unix user with the
`chsh` command. Just tell it what shell program you want to use (e.g. `bash`
or `zsh`) and which user the change is for:

```
$ [sudo] chsh -s /usr/bin/zsh username
```

This command needs to be invoked with root privileges.

This command updates the entry for that user in the `/etc/passwd` file.

[source](http://superuser.com/questions/46748/how-do-i-make-bash-my-default-shell-on-ubuntu)

# Check If A Port Is In Use

The `lsof` command is used to *list open files*. This includes listing
network connections. This means I can check if a particular port is in use
and what process is using that port. For instance, I can check if my rails
application is currently running on port 3000.

```
$ lsof -i TCP:3000
COMMAND   PID       USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
ruby    13821 jbranchaud   12u  IPv6 0xdf2e9fd346cc12b5      0t0  TCP localhost:hbci (LISTEN)
ruby    13821 jbranchaud   13u  IPv4 0xdf2e9fd33ca74d65      0t0  TCP localhost:hbci (LISTEN)
```

I can see that a ruby process (my rails app) is using port 3000. The PID
and a number of other details are included.

See more details with `man lsof`.

h/t [Mike Chau](https://twitter.com/money_mikec)

# Clear The Screen

If you type `clear` into your shell, the screen will be cleared. There is a
handy keybinding though that will save you a few keystrokes. Just hit
`ctrl-l` to achieve the same effect.

source: [Derek P.](https://twitter.com/DerkTheDaring)

# Copying File Contents To System Paste Buffer

If you need to copy and paste the contents of a file, the `pbcopy` command
can be one of the best ways to accomplish this. Simply `cat` the file and
pipe that into `pbcopy` to get the contents of the file into the system
paste buffer.

```
$ cat some-file.txt | pbcopy
```

See `man pbcopy` for more details.

# Create A File Descriptor with Process Substitution

Process substitution can be used to create a file descriptor from the
evaluation of a shell command. The syntax for process substitution is
`<(LIST)` where `LIST` is one or more bash commands.

```
$ cat <(echo 'hello, world')
hello, world
```

This is particularly useful for commands that expect files, such as diff:

```
$ diff <(echo 'hello, world') <(echo 'hello, mars')
1c1
< hello, world
---
> hello, mars
```

Sources: [Brian Dunn](https://twitter.com/higgaion) and
[Bash Guide for Beginners](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_04.html#sect_03_04_07)

# Do Not Overwrite Existing Files

When using the `cp` command to copy files, you can use the `-n` flag to make
sure that you do not overwrite existing files.

# File Type Info With File

Use the `file` utility to determine the type of a file:

```bash
$ file todo.md
todo.md: ASCII English text

$ file Hello.java
Hello.java: ASCII C++ program text

$ file Hello.class
Hello.class: compiled Java class data, version 52.0
```

The `Hello.java` file isn't exactly a C++ program, but close enough.

# Find Newer Files

Use the `-newer` flag with the name of a file to find files that have a
newer modification date than the named file.

For instance,

```
$ find blog -name '*.md' -newer blog/first-post.md
```

will find all markdown files in the `blog` directory that have a
modification date more recent than `blog/first-post.md`.

# Global Substitution On The Previous Command

Let's say we just executed the following command:

```bash
$ grep 'foo' foo.md
```

It gave us the information we were looking for and now we want to execute
a similar command to find the occurrences of `bar` in `bar.md`. The `^`
trick won't quite work here.

```bash
$ ^foo^bar<tab>
$ grep 'bar' foo.md
```

What we need is a global replace of `foo` in our previous command. The `!!`
command can help when we sprinkle in some `sed`-like syntax.

```bash
$ !!gs/foo/bar<tab>
$ grep 'bar' bar.md
```

For a short command like this, we haven't gained much. However, for large
commands that span the length of the terminal, this can definitely save us
a little trouble.

# Hexdump A Compiled File

The `hexdump` unix utility allows you to dump the contents of a
compiled/executable file in a _readable_ hexadecimal format. Adding the `-C`
flag includes a sidebar with a formatted version of that row of hexadecimal.

For example, a compiled _Hello World_ java program, `Hello.java`, will look
something like this:

```
> cat Hello.class | hexdump -C
00000000  ca fe ba be 00 00 00 34  00 1d 0a 00 06 00 0f 09  |.......4........|
00000010  00 10 00 11 08 00 12 0a  00 13 00 14 07 00 15 07  |................|
00000020  00 16 01 00 06 3c 69 6e  69 74 3e 01 00 03 28 29  |.....<init>...()|
00000030  56 01 00 04 43 6f 64 65  01 00 0f 4c 69 6e 65 4e  |V...Code...LineN|
00000040  75 6d 62 65 72 54 61 62  6c 65 01 00 04 6d 61 69  |umberTable...mai|
00000050  6e 01 00 16 28 5b 4c 6a  61 76 61 2f 6c 61 6e 67  |n...([Ljava/lang|
00000060  2f 53 74 72 69 6e 67 3b  29 56 01 00 0a 53 6f 75  |/String;)V...Sou|
00000070  72 63 65 46 69 6c 65 01  00 0a 48 65 6c 6c 6f 2e  |rceFile...Hello.|
00000080  6a 61 76 61 0c 00 07 00  08 07 00 17 0c 00 18 00  |java............|
00000090  19 01 00 0d 48 65 6c 6c  6f 2c 20 57 6f 72 6c 64  |....Hello, World|
000000a0  21 07 00 1a 0c 00 1b 00  1c 01 00 05 48 65 6c 6c  |!...........Hell|
000000b0  6f 01 00 10 6a 61 76 61  2f 6c 61 6e 67 2f 4f 62  |o...java/lang/Ob|
000000c0  6a 65 63 74 01 00 10 6a  61 76 61 2f 6c 61 6e 67  |ject...java/lang|
000000d0  2f 53 79 73 74 65 6d 01  00 03 6f 75 74 01 00 15  |/System...out...|
000000e0  4c 6a 61 76 61 2f 69 6f  2f 50 72 69 6e 74 53 74  |Ljava/io/PrintSt|
000000f0  72 65 61 6d 3b 01 00 13  6a 61 76 61 2f 69 6f 2f  |ream;...java/io/|
00000100  50 72 69 6e 74 53 74 72  65 61 6d 01 00 07 70 72  |PrintStream...pr|
00000110  69 6e 74 6c 6e 01 00 15  28 4c 6a 61 76 61 2f 6c  |intln...(Ljava/l|
00000120  61 6e 67 2f 53 74 72 69  6e 67 3b 29 56 00 20 00  |ang/String;)V. .|
00000130  05 00 06 00 00 00 00 00  02 00 00 00 07 00 08 00  |................|
00000140  01 00 09 00 00 00 1d 00  01 00 01 00 00 00 05 2a  |...............*|
00000150  b7 00 01 b1 00 00 00 01  00 0a 00 00 00 06 00 01  |................|
00000160  00 00 00 01 00 09 00 0b  00 0c 00 01 00 09 00 00  |................|
00000170  00 25 00 02 00 01 00 00  00 09 b2 00 02 12 03 b6  |.%..............|
00000180  00 04 b1 00 00 00 01 00  0a 00 00 00 0a 00 02 00  |................|
00000190  00 00 03 00 08 00 04 00  01 00 0d 00 00 00 02 00  |................|
000001a0  0e                                                |.|
000001a1
```

# Kill Everything Running On A Certain Port

You can quickly kill everything running on a certain port with the following
command.

```bash
sudo kill `sudo lsof -t -i:3000`
```

This gets a list of pids for all the processes and then kills them.

[source](http://stackoverflow.com/questions/9346211/how-to-kill-a-process-on-a-port-on-ubuntu)

# Killing A Frozen SSH Session

Whenever an SSH session freezes, I usually mash the keyboard in desperation
and then kill the terminal session. This can be avoided though. SSH will
listen for the following kill command:

```
~.<cr>
```

This will kill the frozen SSH session and leave you in the terminal where
you were before you SSH'd.

source: [Jack C.](http://hashrocket.com/team/jack-christensen)

# List All The Say Voices

The `say` command can be a fun party trick.

```bash
$ say Get ready for the bass to drop
```

Your friends will be even more impressed when you use some of the alternate
voices.

```bash
$ say -v Daniel Would you like a cup of tea?
```

To see all the alternate voices available, type the following

```bash
$ say -v '?'
```

[source](http://stackoverflow.com/questions/1489800/getting-list-of-mac-text-to-speech-voices-programmatically)

# List All Users

On unix-based systems, all system users are listed with other relevant
information in the `/etc/passwd` file. You can output a quick list of the
users by name with the following command:

```
$ cut -d: -f1 /etc/passwd
```

[source](http://askubuntu.com/questions/410244/a-command-to-list-all-users-and-how-to-add-delete-modify-users)

# Only Show The Matches

Tools like `grep`, `ack`, and `ag` make it easy to search for lines in a
file that contain certain text and patterns. They all come with the `-o`
flag which tells them to only show the part that matches.

This is particularly powerful when used with regex and piped into other
programs.

# Repeat Yourself

Use the `repeat` command to repeat some other command.

You can repeat a command any number of times like so

```
$ repeat 5 say Hello World
```

# Saying Yes

Tired of being prompted for confirmation by command-line utilities? Wish you
could blindly respond 'yes' to whatever it is they are bugging you about?
The `yes` command is what you've been looking for.

```
$ yes | rm -r ~/some/dir
```

This will respond `y` as `rm` asks for confirmation on removing each and
every file in that directory.

`yes` is just as good at saying *no*. Give it `no` as an argument and it
will happily (and endlessly) print `no`.

```
$ yes no
```

# Search History

Often times there is a very specific command you have entered into your bash
prompt that you need to run again. You don't want to have to type it again
and stepping manually through your history may be suboptimal if you typed it
quite a while ago. Fortunately, there is a simple history search feature
that you can use in this kind of situation.

Hit `Ctrl+r` and then start typing a moderately specific search term. Your
search history will be filtered by that term. Subsequent hitting of
`Ctrl+r` will step forward through that filtered history. Once you find the
command you are looking for, hit enter to execute it.

# Search Man Page Descriptions

You can use the `apropos` command with a keyword argument to search for that
words occurrence throughout all the man pages on your system. For instance,
invoking `apropos whatis` will bring up a list of entries including the
`apropos` command itself.

# Securely Remove Files

If you really want to make sure you have wiped a file from your hard drive,
you are going to want to use `srm` instead of `rm`. The man page for `srm`
gives the following description:

> srm  removes  each  specified file by overwriting, renaming, and truncating
> it before unlinking. This prevents other people from undeleting or
> recovering any information about the file from the command line.

# SSH With A Specific Key

When you SSH into another machine using public key authentication, the
key pair from either `~/.ssh/id_dsa`, `~/.ssh/id_ecdsa`, or `~/.ssh/id_rsa`
is used by default. This is generally what you want. But what if the target
server is expecting to identify you with a different SSH key pair?

The `-i` option can be used with `ssh` to specify a different _identity
file_ when the default isn't what you want.

# SSH With Port Forwarding

Use the `-L` flag with `ssh` to forward a connection to a remote server

```
$ ssh someserver -L3000:localhost:3000
```

# Watch The Difference

The `watch` command is a simple way to repeatedly run a particular command.
I'll sometimes use it to monitor the response from some endpoint. `watch`
can make monitoring responses even easier when the `-d` flag is employed.
This flag instructs `watch` to highlight the parts of the output that are
*different* from the previous run of the command.

So if I run

```
$ watch -d curl -LIs localhost:3000
```

I can easily see if the http status of the request changes.

# Watch This Run Repeatedly

I usually reach for a quick bash for loop when I want to run a particular
process a bunch of times in a row. The `watch` command is another way to
run a process repeatedly.

```
watch rspec spec/some/test.rb
```

The default is 2 seconds in between subsequent executions of the command.
The period can be changed with the `-n` flag though:

```
watch -n 2 rspec spec/some/test.rb
```

# Where Are The Binaries?

When I want to know where an executable is, I use `which` like so:

```
$ which rails
/Users/jbranchaud/.gem/ruby/2.1.4/bin/rails
```

That is the rails binary on my path that will be used if I enter a rails command.

However, with something like rails, there may be multiple versions on your
path. If you want to know where all of them are, you can use `where`, like
so:

```
$ where rails
/Users/jbranchaud/.gem/ruby/2.1.4/bin/rails
/Users/jbranchaud/.rubies/2.1.4/bin/rails
/usr/bin/rails
```
