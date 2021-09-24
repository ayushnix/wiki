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
    all of them although scripts written for POSIX `/bin/sh` should probably run reliably across all
    of these platforms.

- shell scripts do not have robust error handling features and they can't handle
  edge cases you didn't think about

    Drew DeVault wrote a [blog post](https://drewdevault.com/2020/12/12/Shell-literacy.html) about
    becoming shell literate. A guy on HN rightfully [pointed
    out](https://news.ycombinator.com/item?id=25402003) the flaws in the operation[^1].

    [Here's](http://typingducks.com/blog/bash/) a blog post about issues when using shell scripts.
    [Here's](https://mywiki.wooledge.org/BashWeaknesses) another page on Greg's Wiki about
    weaknesses in bash.

However, if you know what you're doing, shell scripting using POSIX sh or bash[^2] can be a powerful
tool in your repertoire. You may or may not have access to Python in your Unix-like platform or in
your container but you will always have a shell.

In case your `/bin/sh` is symlinked to `/bin/bash` (Arch Linux), install
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

We'll use our own version of the shell "strict mode" with reasons explained below.

=== "`/bin/sh`"
    ```sh
    set -u

    readonly PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin"
    export PATH
    umask 077

    trap '...' EXIT
    ```

=== "`/bin/bash`"
    ```sh
    set -uo pipefail

    readonly PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin"
    export PATH
    umask 077

    trap '...' EXIT
    ```

- use `#!/bin/sh` (or `#!/bin/bash`) as the shebang

    If a platform doesn't have `/bin/sh` or `/bin/bash` (NixOS, BSDs), adapt your scripts at the
    time of installation.

    Although `#!/usr/bin/env` is better for portability, we'll prefer not to use it. It allows the
    possibility of using arbitrary binaries available in `PATH`. I guess one has bigger problems to
    worry about if the `PATH` on his system can't be trusted. Then again, your scripts may not
    always be used on your system.

- explicity define and export `$PATH` in your script

    This is sort of a follow up of the previous point about not using `#!/usr/bin/env`. We'll
    explicitly define `PATH` in our scripts to make sure that we're using software inside the `/usr`
    directories and not some random software anywhere else on the filesystem.

- set the `umask` to `077` to prevent other users from being able to read/write/execute the
  temporary files that your script might create

We can use `set -e` in `/bin/sh` and `set -Ee` combined with `trap '...' ERR` in `/bin/bash`.
However, we'll do so in isolated blocks of code where external commands aren't involved and where
the behavior of `set -e` is somewhat predictable. We'll need to remember to reset our `ERR` trap
using `trap - ERR` when we do use it.

This section should probably be enough to convince someone that error handling in shell scripting
shouldn't always be relied upon. If you are writing serious scripts or programs that need robust
error handling, you may not want to use shell scripts.

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
would stop at `grep second` but it doesn't. An empty file `sample.txt` is still created. Adding `set
-e` in this case would stop the script after all the commands in the pipe are executed and the next
`echo` command won't be executed.

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

Using `set -C` is going to come down to personal preference. I don't see much point in using it
myself but if you want to, you can use it. If you do use it and need to overwrite an existing file,
either use `>|` or, better yet, redirect the command in question to a temporary file and copy that
file over to the intended file.

### trap

Using `trap` can be helpful if you want to implement cleanup jobs before a script exits. A simple
use case can be deleting temporary files and directories created as part of the script.

Here's a good example to show how `trap` works with `ERR`, `EXIT`, `exit 1`, and `return 1`.

???+ warning
    The `ERR` signal doesn't work on POSIX sh

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
anything `set -e` does. As a result, it might be wiser to not use `set -E` and `trap '...' ERR`
globally and, instead, use them in specific sections and then disable them using `set +E` and `trap
- ERR`.

[This](http://redsymbol.net/articles/bash-exit-traps/) blog post has more details about `trap`.

### errtrace

The usage of `set -E` is only relevant when pairing it with `trap '...' ERR`. From `man bash`,

> If set, any trap on `ERR` is inherited by shell functions, command substitutions, and commands
> executed in a subshell environment.  The `ERR` trap is normally not inherited in such cases.

???+ warning
    `set -E` doesn't work on POSIX sh

## Style Guide

[Google's Shell Style Guide](https://google.github.io/styleguide/shellguide.html) should serve as a
good reference although I don't agree with everything mentioned in it. There's also the [ChromiumOS
Shell Style Guide](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/styleguide/shell.md).

In addition to what's written in the links mentioned above,

- if you don't need to use arrays, write POSIX compliant `/bin/sh` shell scripts

    This should serve as a good rule of thumb. Use `#!/bin/sh` if you don't need arrays and you're
    writing trivial shell scripts. If you need arrays, either use `#!/bin/bash` or a better
    programming language[^3].

- don't use external programs like `sed`, `awk`, `tr`, `cut`, `find` if you don't need to

    One of the primary issues with writing shell scripts is that they lack robust error handling.
    `set -e` isn't really a solution and our so-called "strict mode" is mostly duct tape. The lack
    of robust error handling is mostly exacerbated by using external programs because `set -e`
    doesn't know what to expect from them. By using external programs, you're also introducing
    dependencies to your shell script which may be unnecessary and probably slowing down your script
    as well.

### Minimalism

Even though the syntax of `bash` is weird, a bit verbose, and sometimes jarring, I still ended up
liking shell scripting because of its flexibility and potential simplicity. It's better to keep
things as minimal and simple as possible when writing shell scripts. Your code should be easy to
read and audit, even if it is a bit verbose at times. Here's a simple example.

```sh
num="${1-}"

if [[ "$num" -eq 5 ]]; then
  printf '%s\n' "$num is equal to 5"
fi

[[ "$num" -eq 5 ]] && printf '%s\n' "$num is equal to 5
```

Yes, both of them do the same thing and the 2nd form is shorter to write. I would, however, prefer
using `if-then-fi` simply because it's much easier to understand, even for a complete beginner.

I probably won't use the following features offered by `bash` because they're either not POSIX sh
compatible, can be easily replicated using other features, not needed, or introduce unhealthy
practices.

- `&> file` and `|& tee`
- `$'...'` ANSI C style quotes
- `;&` and `;;&` terminators in `case`
- heredocs (use `printf`; even though it'll be verbose, you gain flexibility with variables)

The list of keywords and built-in commands which I WON'T use are

- `test` (use `[` in `/bin/sh` and `[[` in `/bin/bash`)
- `echo` (use `printf`)
- `select` (use `while`, `case`, `if`)
- `until` (use `while`)
- `function` (completely useless, no idea why anyone would use this)
- `eval` (security issues)
- `typeset` (`declare` seems to be more widely used)
- `readarray` (`mapfile` seems to be more widely used)
- `let` (use `((` and `$((`)

[^1]:
C'mon, who uses white spaces in file and directory names in Linux? Okay, I know I don't but not
everyone is averse to using white spaces in file and directory names, especially people who come
from a Windows background.

[^2]:
You can still write sh compatible scripts while using bash if you use `set -o posix`. Or, just
install dash, symlink `/bin/sh` to it, and use `#!/bin/sh` although this may not work well on your
system. I've done this on Arch Linux and things have been working fine ... so far.

[^3]:
I guess "complex" is a subjective word in this case. In my opinion, any script you personally
consider to be remotely serious should probably be written in another language like Python. But hey,
there's projects like
[password-store](https://git.zx2c4.com/password-store/tree/src/password-store.sh) and
[neofetch](https://github.com/dylanaraps/neofetch/blob/master/neofetch) out there. One of them is a
password manager (well, it uses `gpg` under the hood but it still wraps the whole thing using a bash
script) and the other has 10k+ LOC.
