# All commits where particular line in file changed
If you want to find all the commits where a particular line in a file has
changed, use

```bash
$ git blame filename
```
That gives commit information for each line in `filename`

# New branch from an existing branch

Sometimes, we want to specifically create a new branch off of another branch.
We can do it like so,

```bash
$ git checkout -b myFeature dev
```
Creates MyFeature branch off dev.
[source](http://stackoverflow.com/questions/4470523/git-create-a-branch-from-another-branch)

# Merging without fast-forward

```bash
$ git merge --no-ff myFeature
```

# Fast-forward Merge
In a fast-forward merge, if the branch being merged has all the changes of the
branch being merged to, then the merging branch and the branch receiving the
the merge will be merged linearly.

```
   | (commit 3)
   | (commit 2)
   / feature (new branch forked off of master. commit 1)
  |
  |
master
```

In the above state, the feature branch is ready to be merged into master, but
since there have been no changes to the master branch, the merge is done
linearly and the new state becomes

```
    master
  | (commit 3)
  | (commit 2)
  | feature (branch merged into master linearly. commit 1)
  |
  |
```

# Accessing A Lost Commit

If you have lost track of a recent commit (perhaps you did a reset), you
can generally still get it back. Run `git reflog` and look through the
output to see if you can find that commit. Note the sha value associated
with that commit. Let's say it is `39e85b2`. You can peruse the
details of that commit with `git show 39e85b2`.

From there, the utility belt that is git is at your disposal. For
example, you can `cherry-pick` the commit or do a `rebase`.

# Amend Author Of Previous Commit

The author of the previous commit can be amended with the following command

```bash
$ git commit --amend --author "Don Draper <ddraper@sterlingcooper.com>"
```

[source](http://stackoverflow.com/questions/750172/change-the-author-of-a-commit-in-git)

# Caching Credentials

When public key authentication isn't an option, you may find yourself typing
your password over and over when pushing to and pulling from a remote git
repository. This can get tedious. You can get around it by configuring git
to cache your credentials. Add the following lines to the `.git/config` file
of the particular project.

```
[credential]
    helper = cache --timeout=300
```

This will tell git to cache your credentials for 5 minutes. Use a much
larger number of seconds (e.g. 604800) to cache for longer.

Alternatively, you can execute the command from the command line like so:

```bash
$ git config credential.helper 'cache --timeout=300'
```

[source](http://git-scm.com/docs/git-credential-cache)

# Checkout Old Version Of A File

When you want to return to a past version of a file, you can reset to a past
commit. When you don't want to abandon a bunch of other changes, this isn't
going to cut it. Another option is to just checkout the particular file as
it was at the time of a past commit.

If the sha of that past commit is `72f2675` and the file's name is
`some_file.rb`, then just use checkout like so:

```
$ git checkout 72f2675 some_file.rb
```

# Checkout Previous Branch

Git makes it easy to checkout the last branch you were on.

```bash
$ git checkout -
```

This is shorthand for `git checkout @{-1}` which is a way of referring to
the previous (or last) branch you were on. You can use this trick to easily
bounce back and forth between `master` and a feature branch.

[source](http://stackoverflow.com/questions/7206801/is-there-any-way-to-git-checkout-previous-branch)

# Clean Out All Local Branches

Sometimes a project can get to a point where there are so many local
branches that deleting them one by one is too tedious. This one-liner can
help:

```
$ git branch --merged master | grep -v master | xargs git branch -d
```

This won't delete branches that are unmerged which saves you from doing
something stupid, but can be annoying if you know what you are doing. If you
are sure you want to wipe everything, just use `-D` like so:

```
$ git branch --merged master | grep -v master | xargs git branch -D
```

[source](https://twitter.com/steveklabnik/status/583055065868447744)

# Clean Up Old Remote Tracking References

After working on a Git-versioned project for a while, you may find that
there are a bunch of references to remote branches in your local repository.
You know those branches definitely don't exist on the remote server and
you've removed the local branches, but
you still have references to them lying around. You can reconcile this
discrepancy with one command:

```bash
$ git fetch origin --prune
```

This will prune all those non-existent remote tracking references which is
sure to clean up your git log (`git log --graph`).

[source](http://stackoverflow.com/a/3184742/535590)

# Delete All Untracked Files

Git provides a command explicitly intended for cleaning up (read: removing)
untracked files from a local copy of a repository.

> git-clean - Remove untracked files from the working tree

Git does want you to be explicit though and requires you to use the `-f`
flag to force it (unless otherwise configured).

Git also gives you fine-grained control with this command by defaulting to
only deleting untracked files in the current directory. If you want
directories of untracked files to be removed as well, you'll need the `-d`
flag.

So if you have a local repository full of untracked files you'd like to get
rid of, just:

```bash
$ git clean -f -d
```

or just:

```bash
$ git clean -fd
```

[source](http://stackoverflow.com/questions/61212/remove-local-untracked-files-from-my-current-git-branch)

# Determine The Hash Id For A Blob

Git's `hash-object` command can be used to determine what hash id will be
used by git when creating a blob in its internal file system.

```bash
$ echo 'Hello, world!' > hola
$ git hash-object hola
af5626b4a114abcb82d63db7c8082c3c4756e51b
```

When a commit happens, git will generate this digest (hash id) based on the
contents of the file. The name and location of the file don't matter, just
the contents. This is the magic of git. Anytime git needs to store a blob,
it can quickly match against the hash id in order to avoid storing duplicate
blobs.

Try it on your own machine, you don't even need to initialize a git
repository to use `git hash-object`.

[source](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf)

# Dry Runs in Git

There are a few commands in git that allow you to do a *dry run*. That is,
git will tell you the effects of executing a command without actually
executing that command.

For instance, if you are clearing out untracked files, you can double check
what files are going to be deleted with the *dry run* flag, like so:

```
$ git clean -fd --dry-run
Would remove tmp.txt
Would remove stuff/
```

Similarly, if you want check which files a commit is going to incorporated,
you can:

```
$ git commit --dry-run --short
M  README.md
A  new_file.rb
```

Try running `git commit --dry-run` (that is, without the `--short` flag).
Look familiar? That is the same output you are getting from `git status`.

# Excluding Files Locally

Excluding (read: ignoring) files that should not be tracked is generally
done by listing such files in a tracked `.gitignore` file. Though it doesn't
make sense to list all kinds of excluded files here. For files that you'd
like to exclude that are temporary or are specific to your local
environment, there is another option. These files can be added to the
`.git/info/exclude` file as a way of ignoring them locally.

Add specific files or patterns as needed to that file and then save it.
Relevant files will no longer show up as untracked files when you `git
status`.

# Find The Initial Commit

By definition, the initial commit in a repository has no parents. You can
exploit that fact and use `rev-list` to find the initial commit; a commit
with no parents.

```bash
$ git rev-list --max-parents=0 HEAD
```

The `rev-list` command lists all commits in reverse chronological order. By
restricting them to those with at most 0 parents, you are only going to get
root commits. Generally, a repository will only have a single root commit,
but it is possible for there to be more than one.

See `man git-rev-list` for more details.

[source](http://stackoverflow.com/questions/5188914/how-to-show-first-commit-by-git-log)

# Get File From Stash Or Any Commit

In git, you can reference a commit SHA or branch to checkout differing
versions of files.

```bash
$ git checkout d3d2e38 -- README.md
```

In the same way, you can snag the version of a file as it existed in a
stash. Just reference the relevant stash.

```bash
$ git checkout stash@{1} -- README.md
```

[source](http://stackoverflow.com/questions/1105253/how-would-i-extract-a-single-file-or-changes-to-a-file-from-a-git-stash)

# Grep Over Commit Messages

The `git log` command supports a `--grep` flag that allows you to do a text
search (using grep, obviously) over the commit messages for that repository.
For the git user that writes descriptive commit messages, this can come in
quite handy. In particular, this can be put to use in an environment where
the standard process involves including ticket and bug numbers in the commit
message. For example, finding bug `#123` can be accomplished with:

```bash
$ git log --grep="#123"
```

See `man git-log` for more details.

# Ignore Changes To A Tracked File

Files that should never be tracked are listed in your `.gitignore` file. But
what about if you want to ignore some local changes to a tracked file?

You can tell git to assume the file is unchanged

```bash
$ git update-index --assume-unchanged <some-file>
```

Reversing the process can be done with the `--no-assume-unchanged` flag.

[source](http://blog.pagebakers.nl/2009/01/29/git-ignoring-changes-in-tracked-files/)

# Intent To Add

Git commands like `git diff` and `git add --patch` are awesome, but their
little caveat is that they only work on files that are currently tracked in
the repository. That means that after working on that new feature for 30
minutes, a `git diff` is only going to show you the changes to existing
files and when you go to start making commits with `git add --patch` you
will again be provided with only part of the picture.

The `git add` command has a flag, `-N`, that is described in the git man
pages:

> Record only the fact that the path will be added later. An entry for the
> path is placed in the index with no content. This is useful for, among other
> things, showing the unstaged content of such files
> with git diff and committing them with git commit -a.

By adding untracked files with the `-N` flag, you are stating your *intent
to add* these files as tracked files. Once git knows that these files are
meant to be tracked, it will know to include them when doing things like
computing the diff, performing an interactive add, etc.

# Interactively Unstage Changes

I often use `git add --patch` to interactively stage changes for a commit.
Git takes me through changes to tracked files piece by piece to check if I
want to stage them. This same interactive _staging_ of files can be used in
reverse when removing changes from the index. Just add the `--patch` flag.

You can use it for a single file

```bash
$ git reset --patch README.md
```

or you can let it operate on the entire repository

```bash
$ git reset --patch
```

This is useful when you've staged part of a file for a commit and then
realize that some of those changes shouldn't be committed.

# Last Commit A File Appeared In

In my project, I have a `README.md` file that I haven't modified in a while.
I'd like to take a look at the last commit that modified it. The `git log`
command can be used here with few arguments to narrow it down.

```
$ git log -1 -- README.md
commit 6da76838549a43aa578604f8d0eee7f6dbf44168
Author: jbranchaud <jbranchaud@gmail.com>
Date:   Sun May 17 12:08:02 2015 -0500

    Add some documentation on configuring basic auth.
```

This same command will even work for files that have been deleted if you
know the path and name of the file in question. For instance, I used to have
an `ideas.md` file and I'd like to find the commit that removed it.

```
$ git log -1 -- docs/ideas.md
commit 0bb1d80ea8123dd12c305394e61ae27bdb706434
Author: jbranchaud <jbranchaud@gmail.com>
Date:   Sat May 16 14:53:57 2015 -0500

    Remove the ideas doc, it isn't needed anymore.
```

# List Filenames Without The Diffs

The `git show` command will list all changes for a given reference
including the diffs. With diffs included, this can get rather verbose at
times. If you just want to see the list of files involved (excluding the
diffs), you can use the `--name-only` flag. For instance,

```
$ git show HEAD --name-only
commit c563bafb511bb984c4466763db7e8937e7c3a509
Author: jbranchaud <jbranchaud@gmail.com>
Date:   Sat May 16 20:56:07 2015 -0500

    This is my sweet commit message

app/models/user.rb
README.md
spec/factories/user.rb
```

# List Most Git Commands

You can list most git commands by using the `-a` flag with `git-help`:

```
$ git help -a
```

[Source](http://stackoverflow.com/questions/7866353/git-list-all-available-commands)

# List Untracked Files

Git's `ls-files` command is a plumbing command that allows you to list
different sets of files based on the state of the working directory. So, if
you want to list all untracked files, you can do:

```
$ git ls-files --others
```

This includes files *ignored* by the `.gitignore` file. If you want to
exclude the ignored files and only see *untracked* and *unignored* files,
then you can do:

```
$ git ls-files --exclude-standard --others
```

There are some other flags worth checking at at `git help ls-files`.

[source](http://stackoverflow.com/questions/2657935/checking-for-a-dirty-index-or-untracked-files-with-git)

# Move The Latest Commit To A New Branch

I sometimes find myself making a commit against the `master` branch that I
intended to make on a new branch. To get this commit on a new branch, one
possible approach is to do a reset, checkout a new branch, and then
re-commit it. There is a better way.

```bash
$ git checkout -b my-new-branch
$ git checkout -
$ git reset --hard HEAD~
```

This makes better use of branches and avoids the need to redo a commit that
has already been made.

Note: The example was against the `master` branch, but can work for any
branch.

# Creating a new local copy of a branch in a remote repository

Say there is a remote branch that you want to track locally, then checkout a new
branch with same name as remote branch and then pull the remote branch with same
name locally

```bash
$ git checkout <branch-name-in-remote-you-want-to-copy>
$ git pull <remote-repo-name> <branch-name-in-remote-you-want-to-copy>
```

# Reference A Commit Via Commit Message Pattern Matching

Generally when referencing a commit, you'll use the SHA or a portion of the
SHA. For example with `git-show`:

```
$ git show cd6a63d014
...
```

There are many ways to reference commits though. One way is via regex
pattern matching on the commit message. For instance, if you recently had a
commit with a typo and you had included *typo* in the commit message, then
you could reference that commit like so:

```
$ git show :/typo
Author: Josh Branchaud
Date: Mon Dec 21 15:50:20 2015 -0600

    Fix a typo in the documentation
...
```

By using `:/` followed by some text, git will attempt to find the most
recent commit whose commit message matches the text. As I alluded to, regex
can be used in the text.

See `$ man gitrevisions` for more details and other ways to reference
commits.

[Source](https://twitter.com/jamesfublo/status/678906346335428608)

# Renaming A Branch

The `-m` flag can be used with `git branch` to move/rename an existing
branch. If you are already on the branch that you want to rename, all you
need to do is provide the new branch name.

```bash
$ git branch -m <new-branch-name>
```

If you want to rename a branch other than the one you are currently on, you
must specify both the existing (old) branch name and the new branch name.

```bash
$ git branch -m <old-branch-name> <new-branch-name>
```

# Resetting A Reset

Sometimes we run commands like `git reset --hard HEAD~` when we shouldn't
have. We wish we could undo what we've done, but the commit we've *reset* is
gone forever. Or is it?

When bad things happen, `git-reflog` can often lend a hand. Using
`git-reflog`, we can find our way back to were we've been; to better times.

```bash
$ git reflog
00f77eb HEAD@{0}: reset: moving to HEAD~
9b2fb39 HEAD@{1}: commit: Add this set of important changes
...
```

We can see that `HEAD@{1}` references a time and place before we destroyed
our last commit. Let's fix things by resetting to that.

```bash
$ git reset HEAD@{1}
```

Our lost commit is found.

Unfortunately, we cannot undo all the bad in the world. Any changes to
tracked files will be irreparably lost.

[source](http://stackoverflow.com/questions/2510276/undoing-git-reset)

# Single Key Presses in Interactive Mode

When staging changes in interactive mode (`git add -p`), you have a number
of options associated with single keys. `y` is *yes*, `n` is *no*, `a` is
*this and all remaining*, and so on.

By default, you have to press *enter* after making your selection. However,
it can be configured for single key presses. Add the following to your git
configuration file to enable single key presses for interactive mode:

```
[interactive]
    singlekey = true
```

[source](https://github.com/hashrocket/dotmatrix/blob/master/.gitconfig#L33-L34)

# Staging Changes Within Vim

I've always used git from the command line, but only recently have I started
to leverage [fugitive.vim](https://github.com/tpope/vim-fugitive) to more
quickly do some common git commands from within vim.

I mostly use *fugitive* to stage changes for committing. To stage entire
files, you can view the repository status, `:Gstatus`, and then navigate up
and down (`k` and `j`) tapping `-` next to the files to be staged (or
unstaged).

I've started to use git's interactive mode for staging changes from the
command line (`git add --patch`) more and more and recently wondered if the
same thing can be accomplished with *fugitive*.

It turns out it's pretty simple to do so. Instead of tapping `-` next to a
file you want to stage, you can tap `p` next to it and you will be
immediately dropped into interactive mode for that file. Prepare the lines
you want to stage (or, again, unstage) and save.

# Staging Stashes Interactively

The `-p` flag can be used with `git stash`, just as it is used with `git add`,
for interactively staging a stash.

So, if you have changes in your working tree that you want to stash mixed
with ones that you want to keep working with, then you can simply do:

```
git stash -p
```

Read more on [interactive
staging](https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging).

# Stashing Only Unstaged Changes

If you have both staged and unstaged changes in your project, you can
perform a stash on just the unstaged ones by using the `-k` flag. The
staged changes will be left intact ready for a commit.

```
$ git status
On branch master
...
Changes to be committed:

    modified:   README.md

Changes not staged for commit:

    modified:   app/models/user.rb

$ git stash -k
Saved working directory and index state ...

$ git status
On branch master
...
Changes to be committed:

    modified:   README.md
```

# Stashing Untracked Files

Normally when stashing changes, using `git stash`, git is only going to
stash changes to *tracked* files. If there are any new files in your project
that aren't being tracked by git, they will just be left lying around.

You might not want these *untracked* files left behind, polluting your
working copy. Perhaps these files change your projects functionality or
affect the outcome of tests. You just want them out of the way, for the
moment at least.

And so along comes the `-u` or `--untracked` flag.

```bash
$ touch new_file.rb
$ git stash
No local changes to stash
$ git stash -u
Saved working directory and index state WIP ...
```

# Untrack A Directory Of Files Without Deleting

In [Untrack A File Without Deleting It](untrack-a-file-without-deleting-it.md),
I explained how a specific file can be removed from tracking without
actually deleting the file from the local file system. The same can be done
for a directory of files that you don't want tracked. Just use the `-r`
flag:

```bash
$ git rm --cached -r <directory>
```

[source](http://stackoverflow.com/questions/1143796/remove-a-file-from-a-git-repository-without-deleting-it-from-the-local-filesyste)

# Untrack A File Without Deleting It

Generally when I invoke `git rm <filename>`, I do so with the intention of
removing a file from the project entirely. `git-rm` does exactly that,
removing the file both from the index and from the working tree.

If you want to untrack a file (remove it from the index), but still have it
available locally (in the working tree), then you are going to want to use
the `--cached` flag.

```bash
$ git rm --cached <filename>
```

If you do this, you may also consider adding that file to the `.gitignore`
file.

[source](http://stackoverflow.com/questions/15027873/untrack-and-stop-tracking-files-in-git)

# Verbose Commit Message

Git allows you to display a *diff* of the staged changes in the commit
message buffer. This gives you the opportunity to carefully craft your
commit message in a way that accurately describes the changes being
committed. To display the diff when committing:

```bash
$ git commit -v
```

# What Is The Current Branch?

This question can be answered with one of git's plumbing commands,
`rev-parse`.

```
$ git rev-parse --abbrev-ref HEAD
```

The `--abbrev-ref` flag tells `git-rev-parse` to give us the short name for
`HEAD` instead of the SHA.

[source](http://stackoverflow.com/a/12142066/535590)

# Whitespace Warnings

You can configure git to warn you about whitespace issues in a file when you
diff it. This is handled by the `core.whitespace` configuration. Add the
following to your `.gitconfig` file:

```
[core]
      whitespace = warn
```

By default, git will warn you of trailing whitespace at the end of a line as
well as blank lines at the end of a file.

# Git Cheat Sheet

## Configure
* `-git config –global user.name "[name]"`
 * Sets the name you want attached to your commit transactions
* `git config –global user.email "[email address]"`
 * Sets the email you want attached to your commit transactions
* `git config –global color.ui auto`
 * Enables helpful colorization of command line output
* `git config –global push.default current`
 * Update a branch with the same name as current branch if no refspec is given
* `git config –global core.editor [editor]`
 * Which editor to use when commit and tag that lets you edit messages
* `git config –global diff.tool [tool]`
 * Specify which command to invoke as the specified tool for git difftool

## Create repositories
* `git init [project-name]`
 * Creates a new local repository with the specified name
* `git clone [url]`
 * Downloads a project nd its entire version history

## Make changes
* `git status`
 * Lists all new or modified files to be committed
* `git status -s`
 * Short view of status
* `git diff`
 * Shows file differences not yet staged
* `git add [file]`
 * Snapshots the file in preparation for versioning
* `git add .`
 * Add all modified files to be commited
* `git add '*.txt'`
 * Add only certain files
* `git add –patch filename.x (or -p for short)`
 * Snapshot only chunks of a file
* `git rm [file]`
 * Tell git not to track the file anymore
* `git diff –staged`
 * Show what has been added to the index via git add but not yet committed
* `git diff HEAD`
 * Shows what has changed since the last commit.
* `git diff HEAD^`
 * Shows what has changed since the commit before the latest commit
* `git diff [branch]`
 * Compare current branch to some other branch
* `git difftool -d`
 * Same as diff, but opens changes via difftool that you have configured
* `git difftool -d master..`
 * See only changes made in the current branch
* `git diff –no-commit-id –name-only –no-merges origin/master...`
 * See only the file names that has changed in current branch
* `git diff –stat`
 * See statistics on what files have changed and how
* `git reset [file]`
 * Unstages the file, but preserves its contents
* `git commit`
 * Record changes to git. Default editor will open for a commit message
* `git commit -m "[descriptive message]"`
 * Records file snapshots permanently in version history
* `git commit –amend`
 * Changing the history, of the HEAD commit

## Group changes
* `git branch`
 * Lists all local branches in the current directory
* `git branch [branch-name]`
 * Create a new branch
* `git checkout [branch-name]`
 * Switches to the specified branch and updates the working directory
* `git checkout -b <name> <remote>/<branch>`
 * Switches to a remote branch
* `git checkout [filename]`
 * Return file to it's previous version, if it hasn’t been staged yet
* `git merge [branch]`
 * Combines the specified branch's history into the current branch
* `git merge –no–ff [branch]`
 * Merge branch without fast forwarding
* `git branch -a`
 * See the full list of local and remote branches
* `git branch -d [branch]`
 * Deletes the specified branch
* `git branch -D [branch]`
 * Hard branch delete, will not complain
* `git branch -m <old name > <new name >`
 * Rename a branch

## Refactor filenames
* `git rm [file]`
 * Deletes the file from the working directory and stages the deletion
* `git rm –cached [file]`
 * Removes the file from version control but preserves the file locally
* `git mv [file-original] [file-renamed]`
 * Changes the file name and prepares it for commit

## Suppress tracking
* .gitignore
 * *.log
 * build/
 * temp-*
 * A text file named .gitignore suppresses accidental versioning of files and paths matching the specified patterns
* `git ls-files –other –ignored –exclude-standard`
 * Lists all ignored files in this project

## Save fragments
* `git stash`
 * Temporarily stores all modified tracked files
* `git stash pop`
 * Restores the most recently stashed files
* `git stash list`
 * Lists all stashed changesets
* `git stash drop`
 * Discards the most recently stashed changeset

## Review history
* `git log`
 * Lists version history for the current branch
* `git log -–follow [file]`
 * Lists version history for a file, including renames
* `git log –pretty=format:"%h %s" –graph`
 * Pretty commit view, you can customize it as much as you want
* `git log –author='Name' –after={1.week.ago} –pretty=oneline –abbrev-commit`
 * See what the author has worked on in the last week
* `git log –no-merges master..`
 * See only changes in this branch
* `git diff [file-branch]...[second-branch]`
 * Shows content differences between two branches
* `git show [commit]`
 * Outputs metadata and content changes of the specified commit
* `git blame [file]`
 * Shows when and by whom each line in a file was added

## Redo commits
* `git reset`
 * Unstage pending changes, the changes will still remain on file system
* `git reset [commit/tag]`
 * Undoes all commits after [commit], preserving changes locally
* `git reset –hard [commit]`
 * Discards all history and changes back to the specified commit

## Synchronize changes
* `git fetch [bookmark]`
 * Downloads all history from the repository bookmark
* `git fetch -p`
 * Update history of remote branches, you can fetch and purge
* `git merge [bookmark]/[branch]`
 * Combines bookmark's branch into current local branch
* `git push`
 * Push current branch to remote branch
* `git push [remote] [branch]`
 * Manually specify remote and branch to use every time
* `git push -u origin master`
 * If a remote branch is not set up as an upstream, you can make it so
* `git pull`
 * Downloads bookmark history and incorporates changes
* `git pull [remote] [branch]`
 * Specify to pull a specific branch
* `git remote`
 * See list of remote repos available
* `git remote -v`
 * Detailed view of remote repos available
* `git remote add [remote] [url]`
 * Add a new remote

