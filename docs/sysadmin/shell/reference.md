---
title: "Reference"
summary: "A reference of useful features in POSIX sh and Bash"
date: 2021-09-25
---

## Redirections

The following table depicts the effect of redirection and piping to a file.

You'll have to excuse usage of `echo` in the table due to issues in space. Don't use `echo`. Use
`printf`.

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
A series[^1] of blog posts have been written as well.

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

## Variable Scope

This should give you a good idea of how variables and their scopes work in shell scripting.

```sh linenums="1"
#!/bin/bash

set -uo pipefail

readonly GLOBALVAR="this is a truly global var"

func_ichi() {
  local TORSRV="this is a local variable"
  readonly WGSRV="global variable inside a function"
  return 0
}

func_ni() {
  # get the WGSRV variable in scope by calling the function
  func_ichi

  # we've assigned default values to these variables by using `${VAR-}`
  # otherwise set -u will make the script exit
  printf '%s\n' "output of local variable is - \"${TORSRV-}\""
  printf '%s\n' "output of global variable inside func_one is - \"${WGSRV-}\""
  printf '%s\n' "output of truly global variable is - \"${GLOBALVAR-}\""
}

func_ni
```

Try commenting out line number 15 and see what happens to get the full picture.

[^1]:
[Part 1](https://catonmat.net/bash-one-liners-explained-part-one)
[Part 2](https://catonmat.net/bash-one-liners-explained-part-two)
[Part 3](https://catonmat.net/bash-one-liners-explained-part-three)
