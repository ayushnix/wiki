---
title: "External Commands"
summary: "How to use external commands effectively when writing shell scripts"
date: 2021-09-04
---

There's a good chance that if you're writing shell scripts, you're using external commands which
aren't built-in bash or sh commands. Although one should try to minimize the usage of any external
commands when writing scripts, sometimes, it can't be helped.

Well, if we're gonna use external commands, might as well use them *correctly*.

---

One of the most frequently used commands inside shell scripts is the `find` command. It's also
available in OpenBSD and Alpine Linux. However, it isn't recommended to use `find` if you want to
work on data in a reliable manner. You should try to restrict usage of `find` to the following cases

- informational purposes to prepare lists of files based on certain filters
- use `-delete` instead of `-exec rm -rf {} +`
- use `-execdir` instead of `-exec`
- use ` find ... -print0 | xargs -0 ...` if you really must use it

[Here's](https://github.com/8go/pass-grave/blob/6f26a1bcc046a5d85fb74a4ab27af5c78e66a932/grave.bash#L160)
an interesting example. We have `find ... -exec rm -rf {} +`. However, this doesn't always work as
expected due to differences in how `find` and `rm` works. `find`, by default, lists files like this

```
./.git
./.git/objects
./.git/objects/74
./.git/objects/74/c880afeeca5898cdhf819f4c2133953bf31fcf
```

However, `rm -rf` works on files in this order

```
./.git/objects/74/c880afeeca5898cdhf819f4c2133953bf31fcf
./.git/objects/74
./.git/objects
./.git
```

Now, apparently, using `{} +` instead of `{} \;` is supposed to be faster and more resilient but
from what I tested, we do see a race condition and end up seeing output saying `no such file or
directory found` when `rm -rf` tries to delete something which has already been deleted. In cases
such as the script I linked above which works on your passwords, unexpected behavior is simply not
acceptable.

As mentioned before, using `-delete` resolves this issue. It should also be noted that `-delete` and
`-execdir` are also available on OpenBSD's version of [find](https://man.openbsd.org/find.1).

[Here's](https://www.gnu.org/software/findutils/manual/html_node/find_html/Security-Considerations-for-find.html)
the original source for this information.

---

If a process is running, do some action

```sh
kill -0 "$PID" && printf '%s\n' "$PID is still running"
```
