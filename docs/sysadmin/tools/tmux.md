---
title: "Tmux"
summary: "An overview of some basic operations in tmux"
date: 2021-01-17
---

# Basics

To find out the default value of an option, use the command

```
$ tmux show -g <option>
```

# Attach and Detach

You can specify session names using

```
$ tmux new -s first
```

You can attach to that particular session later using

```
$ tmux attach -t first
```

To create a new session,

```
$ tmux new -d -P -s 1
```

or simply use `:new` from the tmux status line.

To cycle between sessions, use `C-s ()`.

To list the available sessions,

```
$ tmux ls
```

To list the available keybindigs, use `C-s ?`.

Use `C-s /` + `<key>` to get a description about what the key does.

# Tabs (or Windows) and Split Terminals (or Panes)

Create new tabs using `C-s c`.

Create a new directionally horizontal split using either

- `C-s %`, or
- `C-s :` + `sp -h`

A directionally vertical split can be made by using either

- `C-s "`, or
- `C-s :` + `sp -v`

You can move through open tabs using `C-s [0-9]`. A custom index can be entered using `C-s '`. Use
`n` to move to the next tab, `p` to go to the previous tab, and `l` to go back and forth.

You can move around panes by using `C-s ←/→/↑/↓`. If you don't want to use arrow keys, press `C-s q`
+ `<0-9>` to select a pane.

You can close the current tab (and all of its panes) by pressing `C-s &`. A single pane can be
closed using `C-s x`.

You can temporarily maximize a pane using `C-s z` and then restore the layout with the same key.

You can move tabs by using `C-s .` and specifying the new index number. Existing index numbers can't
be specific though.
