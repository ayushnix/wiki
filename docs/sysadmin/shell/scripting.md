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
    set -u

    trap '...' EXIT
    ```

=== "`/bin/bash`"
    ```sh
    set -uo pipefail

    trap '...' EXIT
    ```

`set -e` will be used sparingly in isolated cases where its behavior can be predicted. We probably
won't use `trap '...' ERR` in our scripts at all.

Why are we not using `set -e` globally? Because, in my opinion, it's better to compensate with
shellcheck and our own error handling code in most cases rather than relying on the false sense of
security offered by `set -e`. `trap '...' ERR` makes things even worse. While you can use `set -e`
and `set +e` to enable it for specific blocks of code, `trap '...' ERR` globally unless you reset
the trap.

This section should probably be enough to convince someone that error handling in shell scripting is
basically broken and shouldn't always be relied upon. If you are writing serious scripts or programs
that need robust error handling, don't use shell scripts.

### errexit

`set -e` is supposed to make a script exit immediately whenever it finds a non-zero exit code.
However, this doesn't always work as expected. This is from `man bash`,

> The shell does not exit if the command that fails is part of the command list immediately
> following a `while` or `until` keyword, part of the test following the if or elif reserved words,
> part of any command executed in a `&&` or `||` list except the command following the final `&&` or
> `||`, any command in a pipeline but the last, or if the command's return value is being inverted
> with `!`.  If a compound command other than a subshell returns a non-zero status because a command
> failed while `-e` was being ignored, the shell does not exit.

Basically, if you're using `set -e` or `trap ERR`, you've gotta keep these caveats in mind.

If that wasn't enough, `set -e` may cause our script to exit when we may not want it to.

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
course, writing `((i = i + 1))` would've been better but we shouldn't have to deal with unexpected
events like this.

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

Using `trap` can be helpful if you want to implement cleanup jobs before a script exits. A simple
use case can be deleting temporary files and directories created as part of the script.

However, we should probably avoid using `trap '...' ERR` because it is subject to the same caveats
as `set -e` and unlike `set -e`, we can't enable it selectively. Once enabled, it works globally
throughout your script and may end up catching errors that you don't want and not catching errors
that you do want.

???+ warning
    `ERR` doesn't work on POSIX sh

Here's a good example to show how `trap` works with `ERR`, `EXIT`, `exit 1`, and `return 1`.

```sh
set -Euo pipefail

# we don't need to check the status code
clean_return() {
    printf '%s\n' 'deleting the namespace -- return' >&2
}

# we need to check the status code to determine if action should be taken
# otherwise, even if script exits normally, this function will execute
# and maybe undo everything unintentionally
clean_exit() {
  if [[ "$?" -ne 0 ]]; then
    printf '%s\n' 'deleting the namespace -- exit' >&2
  fi
}

trap clean_return ERR
trap clean_exit EXIT

# triggers the ERR trap
test_err() {
  printf '%s\n' 'begin return'
  if true; then
    return 1                  # return 0 does NOT trigger the ERR trap
  fi
  printf '%s\n' 'end return'
}

# triggers the EXIT trap
test_exit() {
  printf '%s\n' 'begin exit'
  if true; then
    exit 1                    # both exit 0 and exit 1 trigger the EXIT trap
  fi
  printf '%s\n' 'end exit'
}

test_err
test_exit
```

As we've said before, although `trap clean_return ERR` catches `return 1`, it will also catch
anything `set -e` was meant to act upon. As a result, it might be wiser to not use `set -E` and
`trap '...' ERR` at all and call the cleanup function manually instead whenever we use `return 1`.

We can also reset traps to their default behavior inside the script by writing `trap - NAMEOFSIG`.

[This](http://redsymbol.net/articles/bash-exit-traps/) blog post has more details about `trap`.

### errtrace

The usage of `set -E` is only relevant when pairing it with `trap '...' ERR`. From `man bash`,

> If set, any trap on `ERR` is inherited by shell functions, command substitutions, and commands
> executed in a subshell environment.  The `ERR` trap is normally not inherited in such cases.

???+ warning
    `set -E` doesn't work on POSIX sh

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

- try not to use external programs like `sed`, `awk`, `tr`, `cut` etc unless absolutely necessary or
  unless you can save yourself a lot of LOC by using them

    One of the primary issues with writing shell scripts is that they lack robust error handling.
    `set -e` isn't really a solution and our so-called "strict mode" is mostly duct tape. The lack
    of robust error handling is mostly exacerbated by using external programs because shell has no
    idea how to handle them. By using external programs, you're also introducing dependencies to
    your shell script which may be unnecessary. There's also the penalty to speed when using
    external programs.

- we'll prefer not to use the following features because they're either not POSIX sh compatible, can
  be easily replicated using other available features, or simply not needed

    - `&> file` and `|& tee`
    - `select` — just use a combination of `while`, `read`, `case`, and `if`
      instead
    - `until` loop
    - `$'...'` ANSI C style quotes
    - `;&` and `;;&` terminators in `case`
    - `function` keyword to define functions
    - `declare ` built in keyword, except when defining an associative array

It's better to keep things as minimal and simple as possible when writing shell scripts. Although
they can be powerful and might seem fun to write (hey, it might be for some people), if your shell
scripts start becoming too complex[^4], you probably wanna rewrite them in a better language like
Python.

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

# Useful Patterns

## Check Dependencies

If you're using shell scripts, you're most likely using commands that aren't built-in commands of
the shell. Therefore, it makes sense to check whether the commands that you'll use are actually
present on the system before executing a script.

=== "Single Dependency"
    ```sh
    if ! command -v fzf > /dev/null 2>&1; then
      printf '%s\n' "the dependency package fzf was not found!" >&2
      exit 1
    fi
    ```

=== "Multiple Dependencies"
    ```sh
    dependencies=("fzf" "rg" "bat" "lsls")
    for d in "${dependencies[@]}"; do
      if ! command -v "$d" > /dev/null 2>&1; then
        printf '%s\n' "the dependency package $d was not found!" >&2
        exit 1
      fi
    done
    ```

DO NOT use the `which` command for this. The reason is pretty simple. `command` is a built-in
command and it's also POSIX compatible. However, `which` is an external command.

[^1]:
C'mon, who uses white spaces in file and directory names in Linux? Okay, I know I don't but not
everyone is averse to using white spaces in file and directory names, especially people who come
from a Windows background.

[^2]:
You can still write sh compatible scripts while using bash if you use `set -o posix`. Or, just
install dash, symlink `/bin/sh` to it, and use `#!/bin/sh` although this may not work well on your
system. I've done this on Arch Linux and things have been working fine ... so far.

[^3]:
[Part 1](https://catonmat.net/bash-one-liners-explained-part-one)
[Part 2](https://catonmat.net/bash-one-liners-explained-part-two)
[Part 3](https://catonmat.net/bash-one-liners-explained-part-three)

[^4]:
I guess "complex" is a subjective word in this case. In my opinion, any script you personally
consider to be remotely serious should probably be written in another language like Python. But hey,
there's projects like
[password-store](https://git.zx2c4.com/password-store/tree/src/password-store.sh) and
[neofetch](https://github.com/dylanaraps/neofetch/blob/master/neofetch) out there. One of them is a
password manager (well, it uses `gpg` under the hood but it still wraps the whole thing using a bash
script) and the other has 10k+ LOC.
