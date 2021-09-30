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
seems to have popularized the so-called "strict mode" in shell scripting.

In my opinion, you should avoid using the shell "strict mode" in your scripts simply because it's
unreliable and not as important as it might seem provided you're using shellcheck and have spent
time going through [Greg's Wiki](https://mywiki.wooledge.org).

This section should probably be enough to convince someone that native error handling in shell
scripting is broken and shouldn't be relied upon. If you are writing serious scripts or programs
that need robust error handling, you may not want to use shell scripts.

### errexit, errtrace, and ERR

`set -o errexit`, or `set -e`, is supposed to make a script exit immediately whenever it finds a
non-zero exit code. However, this doesn't always work as expected. The usage of `set -e` comes with
a lot of caveats that must be kept in mind and if that wasn't enough, there's no telling what'll
happen when using `set -e` in a script which uses external commands.

This is from `man bash`,

> The shell does not exit if the command that fails is part of the command list immediately
> following a `while` or `until` keyword, part of the test following the if or elif reserved words,
> part of any command executed in a `&&` or `||` list except the command following the final `&&` or
> `||`, any command in a pipeline but the last, or if the command's return value is being inverted
> with `!`.  If a compound command other than a subshell returns a non-zero status because a command
> failed while `-e` was being ignored, the shell does not exit.

```sh
set -e

count=$(grep -c some-string /usr/bin/pass) # script execution aborts here

if [ "${count}" -ne 0 ]; then
  printf '%s\n' "found something"
else
  printf '%s\n' "nothing found"
fi
```

If `grep` fails to find any match, the exit code becomes non-zero and the scripts aborts execution
even if we may not intend to.

There are [plenty](http://typingducks.com/blog/bash/) of other
[examples](https://mywiki.wooledge.org/BashFAQ/105) of unintended behavior when using `set -e`.

`set -o errtrace`, or `set -E` is useful only if we're using `trap '..' ERR`. However, the `ERR`
trap has the same caveats that `set -e` comes with.

Unless you know what you're doing, don't use `set -e`, `set -E`, and `trap '...' ERR`.

???+ warning
    `set -E` and `ERR` don't work on POSIX sh

### pipefail

???+ warning
    `set -o pipefail` doesn't work on POSIX sh

Normally, the exit status of a pipeline is the exit status of the last command in the pipe,
regardless of the exit status of the commands that appeared before it. When we enable `set -o
pipefail` in `/bin/bash`, the exit status of the pipe is the exit status of the last command that
failed.

However, that's all that `set -o pipefail` does. In isolation, it **DOES NOT** cause the script
execution to stop. It simply changes the exit code of the pipe in question and allows the script
execution to go ahead. Only when it's coupled with `set -e` or `|| exit 1` does the script execution
actually stop.

Even if we do use `set -eo pipefail`, all the commands in a pipe are still executed, even if the
first command in a pipe fails. Yes, you read that right. Using `set -e` just doesn't allow the
script to proceed further, but it doesn't actually terminate the script execution in the middle of a
pipe.

Here's a simple, albeit stupid, example to show what we're talking about.

```sh
set -o pipefail

printf '%s\n' "first commmand" | grep second | tee sample.txt
printf '%s\n' "the exit status of the last command was $?"
```

Normally, what you'd expect in this case after using `set -o pipefail` is that the script execution
would stop at `grep second` but it doesn't. An empty file `sample.txt` is still created. Adding `set
-e` in this case would stop the script after all the commands in the pipe are executed and the next
`printf` command won't be executed.

Unfortunately, just like `set -e`, `set -o pipefail` has its own
[quirks](https://mywiki.wooledge.org/BashPitfalls#pipefail) that make it unsuitable for enabling
globally, especially when dealing with
[buffered](https://www.pixelbeat.org/programming/stdio_buffering/)
[output](https://www.perkin.org.uk/posts/how-to-fix-stdio-buffering.html).

### nounset

`set -u` treats unset variables and parameters, except `"$@"` and `"$*"`, as an error and causes the
script to exit. If we use `set -u` and write `$1` and it turns out to be unset, our script will
exit.

Unlike `set -e` and `set -o pipefail`, `set -u` is relatively harmless but it still has its quirks.
If you want to use `set -u`, you'll need to ensure that you

- don't use an ancient version of bash
- use `"${1-}"`, which is relatively uglier, instead of `"$1"`

If you're okay with the tradeoff of making your code less readable, go ahead and use `set -u`. The
alternative is to use shellcheck and test your scripts before deployment.

### trap

A `trap` is typically used for implementing cleanup and fallback jobs. It "traps" a signal and does
something else than what the signal would've done.

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
anything `set -e` does. As a result, it's better to not use

```
set -E
trap '...' ERR
```
globally and, instead, use them in specific sections of the code and then disable them using

```
set +E
trap - ERR
```

[This](http://redsymbol.net/articles/bash-exit-traps/) blog post has more details about `trap`.

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

- do not use all upper case variables unless they're environment variables

    You never know if the all upper case variable you're using is actually a pre-existing
    environment variable.

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
- heredocs for printing help menu (use `printf`; even though it'll be verbose, you gain flexibility
  with variables)

The list of keywords and built-in commands which I WON'T use are

- `test` (use `[` in `/bin/sh` and `[[` in `/bin/bash`)
- `echo` (use `printf`)
- `select` (use `while`, `case`, `if`)
- `until` (use `while`)
- `function` (completely useless, no idea why anyone would use this)
- `eval` (security issues)
- `declare` (`typeset` seems to be more widely supported)
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
script) and the other has 10k+ LOC. [Kubernetes](https://github.com/kubernetes/kubernetes) is
another great example which uses a significant amount of shell script code.
