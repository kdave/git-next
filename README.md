Git branch integration helper.

*Status:* raw, feature incomplete, unpolished, works for me to
assemble for-next branches for linux kernel.

## Syntax

```shell
 git-next [options] [targets...]
```

The result is series of shell commands that lead to the given target's branch
name after merging all depending branches. Feed the instructions to `sh -ex`
that will proceed until the first problem that's not handled internally.

The '-x' switch will show the commands as they're executed, `-e` will stop
execution if any git command fails. Typical non-error failures like deleting
nonexistent branches are handled.

### Options

* `--debug`: print debugging messages as shell comments
* `--dump-config`: parse config sections, variables and branches from the config
* `--list`: list just the sections
* `--fetch`: attempt to fetch before working with a remote
* `--recursive`: recreate all targets recursively
* `--upstream`: change default upstream branch name (default: *master*)

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

### Commands

Third type of argument can be a *command* that can do a separate action on
existing branches. Usually translates to a git command with substituted branch
names or remotes etc.

# Use case

TBD

# About

There are similar tools, but they track the branch assembly instructions in a
way that was (for me) inconvienient to maintain. Previous version was a mockery
written in shell which was hard to extend.

License: [GPL 2](https://www.gnu.org/licenses/gpl-2.0.html)
