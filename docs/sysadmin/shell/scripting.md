---
title: "Shell Scripting"
summary: "An overview of POSIX sh and bash"
date: 2021-01-03
---

# Introduction

Knowing how to use your shell and being able to write scripts that work across different Unix-like
operating systems can be an incredibly important skill. I've often thought about it as an essential
skill that a sysadmin should possess. However, there are various complications when it comes to
using shell scripts.

- although bash is the most popular shell out there, it isn't the standard shell
  used across all Unix-like systems

    Debian uses dash as its `/bin/sh` and Alpine, used widely in containers, uses BusyBox which uses
    its own version of [ash](https://en.wikipedia.org/wiki/Almquist_shell). There's `tcsh` used by
    FreeBSD and `pdksh` used by OpenBSD. Needless to say, there are syntactical differences between
    all of them although scripts written in POSIX sh should probably run reliably across all of
    these platforms.

- shell scripts do not have robust error handling features and they can't handle
  edge cases you didn't think about

    Drew DeVault wrote a [blog post](https://drewdevault.com/2020/12/12/Shell-literacy.html) about
    becoming shell literate. A guy on HN rightfully [pointed
    out](https://news.ycombinator.com/item?id=25402003) the flaws in the operation[^1].

    [Here's](http://typingducks.com/blog/bash/) a blog post about issues when using shell scripts.

However, if you know what you're doing, shell scripting using POSIX sh or bash[^2] can be an
incredibly powerful tool in your repertoire. You may or may not have access to Python in your
Unix-like platform or in your container but you will always have a shell.

We'll talk about POSIX `sh` first and then we'll move on to features specific to `bash`. In case
your `/bin/sh` is symlinked to `bash`, install
[dash](https://git.kernel.org/pub/scm/utils/dash/dash.git). There's also
[mksh](http://www.mirbsd.org/mksh.htm) which seems to be POSIX compliant, a better interactive shell
than dash, and is also used on Android.

## Linter, Formatter, and Testing

Before we get started with bash shell scripting, install
[shellcheck](https://github.com/koalaman/shellcheck). Seriously, don't write shell scripts without
this tool.

We'll also use [shfmt](https://github.com/mvdan/sh) to follow a specific style guide and stick to
it. I'm using `-i 2 -ci -sr -bn` options with shfmt.  You'll need a plugin like
[ALE](https://github.com/dense-analysis/ale) to use shellcheck and shfmt on neovim.

I'm not sure if [bats](https://github.com/bats-core/bats-core) is helpful. There's also
[shellspec](https://github.com/shellspec/shellspec) and [shunit2](https://github.com/kward/shunit2).

## Shell Strict Mode

[This](http://redsymbol.net/articles/unofficial-bash-strict-mode/) blog post, for better or worse,
seems to have popularized strict mode in shell scripting.

We'll use our own version of the unofficial strict mode with reasons explained below.

=== "`/bin/sh`"
    ```sh
    set -Eu
    trap 'echo "..." >&2' ERR

    set -e
    ...
    ...
    set +e
    ```

=== "`/bin/bash`"
    ```sh
    set -Euo pipefail
    trap 'echo "..." >&2' ERR

    set -e
    ...
    ...
    set +e
    ```

`set -e` will be used sparingly in cases where unexpected input/output can't be tolerated and in
cases where we'd want the script to exit in case of errors.

### errexit

`set -e` is supposed to make a script exit immediately whenever it finds a non-zero exit code.
Exceptions include commands executed as part of `while`, `if`, `elif`, and commands in `&&`, `||`,
and `|` except the last command.

However, this doesn't always work as expected. `set -e` may cause our script to exit when we may not
want it to.

```sh
set -e

count=$(grep -c some-string /usr/bin/pass) # script execution aborts here

if [ "${count}" -ne 0 ]; then
  printf '%s\n' "found something"
else
  printf '%s\n' "nothing found"
fi
```

Here's another example.

```sh
set -e

i=0
((i++)) # script execution aborts here
echo "$i"
```

In the first case, if `grep` fails to find any match, the exit code becomes non-zero. However,
that's fine, because we intend to use it as a test case. But using `set -e` aborts execution. In the
second case, the shell exits when it finds that the value of the arithmetic expression is 0. Of
course, the better method here is to write `((i = i + 1))` instead.

There are [plenty](http://typingducks.com/blog/bash/) of other
[examples](https://mywiki.wooledge.org/BashFAQ/105) of unintended behavior when using `set -e`. As a
result, **we WON'T be using** `set -e` in our scripts, at least not globally.

### nounset

Unlike `set -e`, `set -u` is relatively harmless and can be useful provided we deal with unset
positional parameters.

```sh
set -u

if [ -n "${1-}" ]; then
  echo "the first parameter wasn't empty and it had ${1-}"
else
  echo "the first parameter was either unset or empty"
fi
```

If we write `$1` and it's unset, `set -u` will cause the script to exit.

Since `set -u` doesn't come with potential false positives (that I know of), **we WILL USE** `set
-u` in our scripts.

### pipefail

???+ warning
    `set -o pipefail` doesn't work on POSIX sh

Normally, the exit status of a pipeline is the exit status of the last command in the pipe. When we
enable `set -o pipefail` in `/bin/bash`, the exit status is that of the last command from the right
which fails.

However, that's all that `set -o pipefail` does. In isolation, it **DOES NOT** cause the script
execution to stop.  It simply changes the exit code of the pipe in question and allows the script
execution to go ahead. Only when it's coupled with `set -e` or `|| exit 1` does the script execution
actually stop.

If we do use `set -e`, all the commands in a pipe are still executed, even if the first command in a
pipe fails. Yes, you read that right. Using `set -e` just doesn't allow the script to proceed
further, but it doesn't actually terminate the script execution at the command which fails in a
pipe.

Here's a simple (albeit stupid) example to show what we're talking about.

```sh
set -o pipefail

echo "first commmand" | grep second | tee sample.txt
echo "the exit status of the last command was $?"
```

Normally, what you'd expect in this case after using `set -o pipefail` is that the script execution
would stop at `grep second` but it doesn't. An empty file `sample.txt` is created and the next
command is also executed. Adding `set -e` in this case would stop the script after all the commands
in the pipe are executed and the next `echo` command won't be executed.

If you weren't aware about this behaviour, it should probably make you see pipes in Linux with a new
perspective. That being said, we **CAN USE** `set -o pipefail` in our `#!/bin/bash` scripts as long
as we're aware of its limitations.

### noclobber

`set -C` prevents scripts from overwriting existing files. The commands in the following script will
run successfully except line number 3 assuming that `sample.txt` already exists.

```sh linenums="1"
set -C

echo "overwrite" > sample.txt
echo "append" >> sample.txt
echo "send to null" > /dev/null
```

If you create the file using `touch sample.txt` before line 3, that will also cause line 3 to fail.
The only case when line 3 would succeed is when `sample.txt` didn't exist before the execution of
line 3.

You can override the effect of `set -C` by using `>|` instead of `>`.

We can use `set -C` in our scripts without issues but using it can be optional I think. It depends
on the person writing the code. When using `set -C`, if you really need to overwrite an existing
file, either use `>|` or, better yet, redirect the command in question to a temporary file and copy
that file over to the intended file.

### trap

Even after using [shellcheck](https://github.com/koalaman/shellcheck) and writing scripts carefully,
you never know when a script might exit abruptly due to unforseen reasons. The exit may not be
informative either. What if you created a bunch of directories and files but before those are
cleaned as intended, the script exits? What if it was an expensive VM or a container?

We can, and should, use `trap` on both `/bin/sh` and `/bin/bash` to catch different exit signals and
execute a command before the script actually comes to a stop. A simple example would be to catch a
silent `ERR`.

The standard format for an `ERR` trap can be as follows

```sh
trap 'echo >&2 "$LINENO: error while executing $BASH_COMMAND"' ERR
```

The commands that should be executed with a trap can also be mentioned in a function

```sh
# $LINENO doesn't give expected results when used inside a function
cleanexit() {
  echo >&2 "$0: error while executing $BASH_COMMAND"
}
trap cleanexit ERR
```

The `ERR` signal behaves much like `set -e` except that it doesn't stop the script execution. For
that, you can use `exit` in the commands passed to the trap. Of course, you can trap other signals
like `INT`, `TERM`, and `EXIT` and execute the tasks that you want.

[This](http://redsymbol.net/articles/bash-exit-traps/) blog post has more details about `trap`.

In order to ensure that the trap works as expected when using functions and subshells, we'll also
use `set -E`.

### errtrace

As mentioned before, **WE'LL USE** `set -E` to make `trap '...' ERR` work when using functions and
subshells. However, unlike `set -e`, `set -E` doesn't make the script exit. It simply allows `trap`
to recognize `ERR` when it wouldn't.

=== "Without `set -E`"
    ```sh
    # the trap is never executed
    trap '"error occured at $BASH_COMMAND"' ERR

    sample() {
      printf '%s\n' 'first line followed by false'
      false
      printf '%s\n' 'second line after false'
    }

    sample
    ```

=== "With `set -E`"
    ```sh
    # the trap is executed when the function is called
    set -E

    trap '"error occured at $BASH_COMMAND"' ERR

    sample() {
      printf '%s\n' 'first line followed by false'
      false
      printf '%s\n' 'second line after false'
    }

    sample
    ```

## Style Guide

[Google's Shell Style Guide](https://google.github.io/styleguide/shellguide.html) should serve as a
good reference.

- use `#!/bin/sh` (or `#!/bin/bash`, if you need bash-isms) as the shebang

    If a platform doesn't have `/bin/sh` or `/bin/bash` (NixOS, BSDs), adapt your scripts
    accordingly at the time of installation.

    Although `#!/usr/bin/env sh` is better for portability, we'll prefer not to use it. You can't
    pass arguments in this form and using it allows the possibility of using arbitrary binaries
    available in `PATH`. I guess one has bigger problems to worry about if the `PATH` on his system
    can't be trusted. Then again, your scripts may not always be used on your system so there's
    that.

- we'll prefer not to use the following features because they're either not POSIX sh compatible, can
  be easily replicated using other available features, or simply not needed

    - `&> file` and `|& tee`

    - `select` — just use a combination of `while`, `read`, `case`, and `if`
      instead

    - `until` loop

    - `$'...'` ANSI C style quotes

    - `;&` and `;;&` terminators in `case`

    - `function` keyword to define functions

# `/bin/bash`

## Redirections

The following table depicts the effect of redirection and piping to a file.

`stdin` has the file descriptor `0`, `stdout` works on `1`, and stderr works on `2`.

| syntax           | terminal stdout | terminal stderr | file stdout | file stderr | effect on file |
| ---------------- | --------------- | --------------- | ----------- | ----------- | -------------- |
| `>`              | no              | yes             | yes         | no          | overwrite      |
| `>>`             | no              | yes             | yes         | no          | append         |
| `2>`             | yes             | no              | no          | yes         | overwrite      |
| `2>>`            | yes             | no              | no          | yes         | append         |
| `> file 2>&1`    | no              | no              | yes         | yes         | overwrite      |
| `>> file 2>&1`   | no              | no              | yes         | yes         | append         |
| `echo "msg" >&2` | no              | yes             | no          | no          | -              |
| `| tee`          | yes             | yes             | yes         | no          | overwrite      |
| `| tee -a`       | yes             | yes             | yes         | no          | append         |
| `2>&1 | tee`     | yes             | yes             | yes         | yes         | overwrite      |
| `2>&1 | tee -a`  | yes             | yes             | yes         | yes         | append         |

You might wonder about the difference between `> file 2>&1` and `2>&1 > file`. In the first case,
you won't get any output on the terminal and both stdout and stderr are redirected to `file`.
However, in the 2nd case, stderr is still shown on the terminal.

For understanding the meaning behind `a>&b`, read this as `a` is now pointing wherever `b` is
pointing at the moment. Also, `a` won't stop pointing wherever `b` was pointing to if `b` starts
pointing anywhere else.

This implies that `> file 2>&1` means that point stdout to `file`, and also point stderr to wherever
stdout is pointing to (which is `file`). Similarly, `2>&1 > file` implies that point stderr to
wherever stdout is poiting to (which is the terminal at this point) and then point stdout to `file`.
Now, stderr doesn't stop pointing to the terminal.

[Here's](https://catonmat.net/ftp/bash-redirections-cheat-sheet.pdf) a cheatsheet that should help.
A series[^3] of blog posts have been written as well.

## Conditional Constructs

The conditional expression `[[ <expression> ]]` returns a value of 0 or 1 depending on the
evaluation of the expression inside.

A list of common tests you can do inside `[[ ... ]]` are

| Operator Syntax        | Description                                                        |
| ---------------------- | ------------------------------------------------------------------ |
| `-e file`              | TRUE, if file exists                                               |
| `-f file`              | TRUE, if a regular file exists                                     |
| `-d file`              | TRUE, if a directory exists                                        |
| `-r, -w, -x file`      | TRUE, if read, write, executable permissions exist                 |
| `-s file`              | TRUE, if size of the file is greater than 0 (not empty)            |
| `file1 -nt file2`      | TRUE, if file1 is newer than file2; use `-ot` for the antonym      |
| `-z string`            | TRUE, if string is empty                                           |
| `-n string`            | TRUE, if string is not empty                                       |
| `string1 == string2`   | TRUE, if string1 matches the **pattern** in string2                |
| `string1 == "string2"` | TRUE, if string1 literally matches string2                         |
| `string1 != string2`   | TRUE, if string1 doesn't match the **pattern** in string2          |
| `string1 =~ string2`   | TRUE, if string1 matches the extended regex pattern in string2     |
| `string1 < string2`    | TRUE, if string1 sorts before string2                              |
| `int1 -eq int2`        | TRUE, if both integers are identical; `-ne` for the antonym        |
| `int1 -lt int2`        | TRUE, if int1 is less than int2; `-gt` for the antonym             |
| `int1 -le int2`        | TRUE, if int1 is less than or equal to int2; `-ge` for the antonym |

[^1]: C'mon, who uses white spaces in file and directory names in Linux? Okay,
I know I don't but not everyone is averse to using white spaces in file and
directory names, especially people who come from a Windows background.

[^2]: You can still write sh compatible scripts while using bash if you use `set
-o posix`. Or, just install dash, symlink `/bin/sh` to it, and use
`#!/bin/sh`.

[^3]: [Part 1](https://catonmat.net/bash-one-liners-explained-part-one), [Part
2](https://catonmat.net/bash-one-liners-explained-part-two), and [Part
3](https://catonmat.net/bash-one-liners-explained-part-three).
