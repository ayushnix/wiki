---
title: "Useful Snippets"
summary: "A list of useful code blocks for use in scripts"
date: 2021-09-25
---

If a process is running, do some action

```sh
if [[ kill -0 "$PID" ]]; then
  printf '%s\n' "$PID is still running"
fi
```

---

If you're using shell scripts, you're most likely using commands that aren't built-in commands of
the shell. Therefore, it makes sense to check whether the commands that you'll use are actually
present on the system before executing a script. Although this is the job of a package manager, it
doesn't hurt to simply not allow running a script in case a dependency isn't present.

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

You can use the shell built-in commands `hash` and `type -p` as well although I'm not sure if they
are POSIX compatible. From what I've tested so far, both of them are available on `dash`, `mksh`,
and busybox `ash`. The `hash` built-in is interesting because it stores the entries you ask for in a
hash table for faster access. It also doesn't append anything to stdout so you can write something
like `hash "$d" 2> /dev/null` instead which is shorter.

**DO NOT** use the `which` command for this. The reason is pretty simple. `command -v` is POSIX
compatible and a shell built in keyword. There's no reason[^1] to use an external command and take on
the extra costs and uncertainties when you don't need to.

[^1]:
There's a minor caveat when using `command -v`. If you use `command -v` interactively and if the
command you're looking for has an alias, `command -v` will print that alias instead of exiting with
an error. So, if you've defined an alias for `ll` and you execute `command -v` interactively, you'll
get `alias ll='ls -lAh'` as the output. This doesn't happen when using `command -v` inside scripts
though so you can still go ahead and use `command -v` instead of `which` when writing scripts.
