Git branch integration helper.

*Status:* feature semi-complete, works for me to assemble for-next branches for
linux kernel.

## Syntax

```shell
 git-next [options] [actions...] [targets...]
```

The result is series of shell commands that lead to the given target's branch
name after merging all depending branches. Feed the instructions to `sh -ex`
that will proceed until the first problem that's not handled internally.

The `-x` switch will show the commands as they're executed, `-e` will stop
execution if any git command fails. Typical non-error failures like deleting
nonexistent branches are handled.

### Options

* `--debug`: print debugging messages as shell comments
* `--dump-config`: parse config sections, variables and branches from the config
* `--list`: list just the sections
* `--tree`: dump the tree structure of the given target(s)
* `--fetch`: attempt to fetch before working with a remote
* `--recursive`: recreate all targets recursively
* `--upstream`: change default upstream branch name (default: *master*)
* `--dry-run`: print the shell commands, do not execute
* `--remote`: comma separated list of remote names, curretnly works with a few
  actions (see bellow), a special remote name `.` means the local and the git
  commands are adjusted accordingly

### Arguments

List of targets (ie. section names), print shell commands to generate given
branches.

## Config

A simple example

```ini
[testme]
NAME=mytest
BASE=master

# my new feature
topic/feature
```

Start with branch `master` and merge `topic/feature` into it, the result will
be called `mytest`.

### Variables

Mandatory:

* `NAME=string`: name of the final branch
* `BASE=string`: starting point
* `MERGEBASE=string`: starting point created as result of `git
  merge-base value $upstream`, where *value* is from config and *$upstream* is
  internally defined as *master*
* `RBRANCH=string`: remote branch as base, also requires `REMOTE`

Only one of *BASE*, *MERGEBASE* or *RBRANCH* is allowed.

Other:

* `TIMESTAMP=bool`: append a timestamp in format *-%Y%m%d* to the branch name
* `SEALED=bool`: do nothing for the given target
* `REMOTE=string`: name of remote to fetch the `RBRANCH`
* `RNAME=string`: name of remote branch to push to, requires `REMOTE`
* `GITMERGEOPTIONS`: additional options to `git-merge`, eg. strategy, squash or
  `--allow-unrelated-histories`
* `HOOKPREPUSH`: run the command before the `git push` and don't push if it fails

User-defined:

* `KEY=value`: the *KEY* must not conflict with the generic variables, value is mandatory

For example `VERSION=1.2.3`, that can be later used in expansion of other
variable: `NAME=mynext-%(VERSION)s` .

### Branches and other targets

The no-value keys of the config refer to other branches or targets. Plain
branches do not start with `@`, otherwise they're considered as references to
other targets in the config.

Target refs are replaced by resolved name of the branch (ie. variable
expansion, timestamp). The included branches are topologically sorted and the
output starts from the lowest dependencies up to the given target. No recursion
is allowed.

Duplicate references are silently ignored, because the ini file parser works
like that.

### Actions

Third type of argument can be a command that can do a separate *action* on
existing branches. Usually translates to a git command with substituted branch
names or remotes etc.

* `+push`: push to remote, translates to `git push REMOTE NAME:RNAME`
* `+pushf`: forced push to remote, translates to `git push -f REMOTE NAME:RNAME`
* `+tslist`: list names of existing local timestamped branches for a given
  target, recursion is supported
* `+tslist1`: dtto, but ommit the last one, typically the branch that's been
  created on the same day
* `+tsclean`: similar to `tslist`, but emit git commands to delete the local
  branch or delete the remotes, recursion is supported, optional is the list of
  remotes
* `+tsclean1`: dtto, but ommit the last one, typically the branch that's been
  created on the same day
* `+checkout` or `+co`: checkout the branch of a given target (eg. with
  timestamp applied)
* `+rebase`: rebase using the expanded target branch name, no options to rebase
  are passed now, use without the execution flag and copy the command line if
  you need interactive rebase etc
* `+fetch`: separate command to fetch all remotes reachable from a given target

# Use case

TBD

# About

There are similar tools, but they track the branch assembly instructions in a
way that was (for me) inconvienient to maintain. Previous version was a mockery
written in shell which was hard to extend.

Hilights:

* the config is in a separate file
* ... can be tracked externally in git as well
* ... sections can be copied, renamed, quick-tested, archived
* ... in-place comments help tracking status
* currently, only the commands are printed
* ... so dangerous operations are not immediatelly destructive
* ... you'll see exactly what's going to be executed
* ... ... and that the git web tools are cool but git command line is really powerful

License: [GPL 2](https://www.gnu.org/licenses/gpl-2.0.html)
