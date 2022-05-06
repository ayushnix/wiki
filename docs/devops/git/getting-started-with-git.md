---
title: "Getting Started With Git"
summary: "Learn how git works by understanding it rather than rote learning the commands."
date: 2020-12-06
---

Instead of rote learning and hoping not to mess up when executing git commands, it's better to
understand what's going on under the hood and how git actually works. This guide isn't meant to be a
traditional git 101 guide. We'll cover basic terminology briefly but focus solely on git internals
because, in my opinion, that's how one should get started with git.

# Introduction

There are several insightful one line definitions of what a git repository is but we'll start with
this one.

> A git repository is a collection of objects and names for those objects called references or,
> simply, refs.

Let's see this for ourselves.

```
$ git init repo
$ cd repo/
$ tree .git/
.git/
├── branches
├── HEAD
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

For now, we are concerned with the `objects/` and `refs/` directories inside `.git/`. Yes, we've
omitted some output from the `tree` command to focus on what's important.

We usually deal with git using its high level porcelain commands rather than its low level plumbing
commands. We'll use the plumbing commands to do a basic operation in git.

## Git Objects

Let's start by adding something to a git repo.

```
$ echo "git internals" | git hash-object -w --stdin
613b11344587e365a3e20af88e29fbaea95cd458
$ tree .git/objects/
.git/objects/
├── 61
│   └── 3b11344587e365a3e20af88e29fbaea95cd458
├── info
└── pack
```

Notice that we didn't even use a file to store content in git, we just `echo`ed content into the
standard input which is received by the `git hash-object` command which then computes the
hexadecimal sha1 hash of the content passed via stdin and stores it in the objects folder.

Oh, if you try to use the `sha1sum` command on the same input, you won't get the same result. Git
appends some meta data about the content that it stores in its `objects/` *key-value store*. You'll
need to prefix the type of the object, its size, and a NULL byte to get the same result. You can get
the type and size of the object you just stored in the git repo using the `git cat-file` command.

```
$ git cat-file -t 613b11
blob
$ git cat-file -s 613b11
14
$ echo -e "blob 14\0git internals" | sha1sum | tr -d '[:blank:]' | tr -d '-'
613b11344587e365a3e20af88e29fbaea95cd458
```

This brings us to the fundamental object used by git to store data — **blobs**.  Git stores the
contents of files in objects called blobs, binary large objects.  The difference between a blob and
a file is that a file also contains meta data.  For example, a file “remembers” when it was created,
so if you move that file into another directory, its creation time remains the same. A blob, on the
other hand, is just content — binary streams of data. A blob doesn't register its creation date, its
name, or anything but its content.

As seen above, we can compute the sha1 hash of the blob that the git computes using the format
`"blob ${#CONTENTS}\0$CONTENTS"`.

What now?

```
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Git doesn't know anything about the object we just created because there's nothing inside our
**working directory**, the directory where your actual content should be. The blob we created is
also not present in the **staging area** aka **index** yet. Let's add it to the staging area.

```
$ git update-index --add --cacheinfo 100644,613b11344587e365a3e20af88e29fbaea95cd458,git-internals.txt
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   git-internals.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    git-internals.txt
```

This is interesting. We've added the file `git-internals.txt` into the staging area but since that
file isn't present in the working directory, git thinks that it has been deleted. However, in
reality, that file has never been there in the working directory to begin with. We'll come back to
this issue later so just forget about it for now. After all, we're dealing with the git repository
here, not its working directory and since a git repository is a *content addressable* storage and a
key-value store, if we have the required objects, we can get the required files.

Remember that we said that blobs are simply content and nothing else? Git stores the meta data of
blobs into another fundamental storage unit called a **tree** object.

```
$ git write-tree
c19878e8ba21805cc916151c15ba14f9e83d2cd9
$ git cat-file -t c19878
tree
$ git cat-file -p c19878
100644 blob 613b11344587e365a3e20af88e29fbaea95cd458	git-internals.txt
```

If you read the man page of `git write-tree`, you'll see that it mentions that it "creates a tree
object using the current index". The tree object stores the meta data of blobs and other trees. This
meta data includes the file mode, type of object, the sha1 hash of the object, and the file or
directory name.

If you notice the contents of the `.git/` directory now, it'll be something like this:

```
$ tree .git/objects/
.git/objects/
├── 61
│   └── 3b11344587e365a3e20af88e29fbaea95cd458
├── c1
│   └── 9878e8ba21805cc916151c15ba14f9e83d2cd9
├── info
└── pack
```

The `objects/` directory now contains another entry for the tree object. At this point, the output
of `git status` will remain unchanged.

Let's go ahead and create another fundamental storage unit, the **commit** object.

```
$ git commit-tree -m "first commit" c19878
cc219e7bb9a8275759ba1bfd7e4510bc2e6ff1d1
$ tree .git/objects/
.git/objects/
├── 61
│   └── 3b11344587e365a3e20af88e29fbaea95cd458
├── c1
│   └── 9878e8ba21805cc916151c15ba14f9e83d2cd9
├── cc
│   └── 219e7bb9a8275759ba1bfd7e4510bc2e6ff1d1
├── info
└── pack
$ git cat-file -t cc219e
commit
$ git cat-file -p cc219e
tree c19878e8ba21805cc916151c15ba14f9e83d2cd9
author Ayush Agarwal <email@email.com> 1608903575 +0530
committer Ayush Agarwal <email@email.com> 1608903575 +0530

first commit
```

As shown in the output of the `git cat-file` command, the commit object stores the meta data of the
tree object, the author, the committer, the time at which the commit was done, and the commit
message. Since this is the first commit in our repository, this commit object doesn't point to a
parent commit object yet.

While a tree represents a particular directory state of the working directory, a commit represents
that state in "time", and explains how to get there. Think of a commit object as a snapshot of your
working directory at a point in time.  When we use the porcelain commands of git, we usually
interact with commit objects and their sha1 hashes.

You might notice something strange though.

```
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   git-internals.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    git-internals.txt
```

`git status` still doesn't think that we've committed anything yet. There's also the issue of the
missing file in our working directory which git thinks has been deleted.

We've finally arrived at the `refs/` folder.

## Git References

Remember that we said that a git repository is a key-value store? Well, the store is the
`.git/objects/` directory and the keys are in the `.git/refs/` directory. In git terminology, values
are objects and *references* are keys.  References can be anything from a local branch, a remote
branch, an immutable tag, and a file called HEAD which stores a *symbolic reference* (reference to
another reference) unless you're in a detached HEAD state and HEAD then contains a sha1 hash as its
reference. This also shows that the sha1 commit hashes can themselves be used as references.

By default, a git repository starts with a local branch called master. Let's see what the
`.git/HEAD` file contains.

```
$ cat .git/HEAD
ref: refs/heads/master
```

The `.git/HEAD` file just contains a symbolic reference to the master branch.  The location of the
master branch should be familiar - it's inside the `.git/refs/heads/` folder. All we need is a file
called `master` which points to the sha1 hash of the commit object that we made a few minutes ago.

```
$ echo "cc219e7bb9a8275759ba1bfd7e4510bc2e6ff1d1" > .git/refs/heads/master
```

Rather than manually adding the sha1 hash of the commit object, we can also use the `git update-ref`
command to change the value of `.git/refs/heads/master` in a safer manner.

```
$ git update-ref refs/heads/master cc219e
```

Finally, we've successfully made a commit object.

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    git-internals.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git log
commit cc219e7bb9a8275759ba1bfd7e4510bc2e6ff1d1 (HEAD -> master)
Author: Ayush Agarwal <email@email.com>
Date:   Fri Dec 25 19:09:35 2020 +0530

    first commit
```

To get a clean `git status` output, we'll add the `git-internals.txt` file by getting it from git to
demonstrate that a git repository ensures the data integrity of your working directory using commit
objects.

```
$ git checkout git-internals.txt
```

This command retrieves the file `git-internals.txt` from the staging area and writes it in the
working directory. We can write a specific commit object hash before the file name to checkout the
version of the file in that commit object.

`git status` is now finally clean and shows us the expected output.

```
$ git status
On branch master
nothing to commit, working tree clean
```

Just for the sake of reference (pun not intended), we would've used the following porcelain commands
to achieve the same result.

```
$ git init repo
$ cd repo/
$ echo "git internals" > git-internals.txt
$ git add git-internals.txt
$ git commit -m "first commit"
```

The `git add` command adds the `git-internals.txt` file to the staging area and creates the blob
object and adds it to the `.git/objects/` folder. The `git commit` command creates a tree object,
points it to the blob object, creates a commit object which points to the tree object it just
created, and updates the `.git/refs/heads/master` file with the sha1 hash of the commit object it
creates.

This is it. You now know the basics of how a git repo actually works under the hood. However, we can
still dive in a bit more.

# Visualizing Git

Let's create another file and another commit object.

```
$ echo "git internals v2" > git-internals-v2.txt
$ git add git-internals-v2.txt
$ git commit -m "second commit"
[master 11e5833] second commit
 1 file changed, 1 insertion(+)
 create mode 100644 git-internals-v2.txt
$ git log --oneline --decorate --graph --all
* 11e5833 (HEAD -> master) second commit
* cc219e7 first commit
```

In simple visual terms, all we've done is create this *directed acyclic graph*:

![Initial Commits - Dark Mode](/images/initial-commits-2x-dark.webp#only-dark)
![Initial Commits - Light Mode](/images/initial-commits-2x-light.webp#only-light)

However, let's see what our repo looks like when we represent all the git objects and references.

![Git Repo - Dark Mode](/images/git-repo-2x-dark.webp#only-dark)
![Git Repo - Light Mode](/images/git-repo-2x-light.webp#only-light)

It should be clear from this diagram that branches are just references that are pointers to commit
objects. This pointer moves automatically throughout the graph as you commit more objects and
perform other operations.

```
$ git checkout -b test
$ git log --oneline --decorate --graph --all
* 11e5833 (HEAD -> test, master) second commit
* cc219e7 first commit
$ echo "branches" >> git-internals.txt
$ git commit -am "third commit"
[test c4691ec] third commit
 1 file changed, 1 insertion(+)
$ git log --oneline --decorate --graph --all
* c4691ec (HEAD -> test) third commit
* 11e5833 (master) second commit
* cc219e7 first commit
```

Now, check the contents of the `.git/refs` directory.

```
$ tree .git/refs/
.git/refs/
├── heads
│   ├── master
│   └── test
└── tags
$ cat .git/refs/heads/test
c4691ec4ee1a3c9fa257cfac16192dbc022c8b9a
```

Where does `HEAD` point to?

```
$ cat .git/HEAD
ref: refs/heads/test
```

As we can see, right now, `HEAD` is symbolic reference to the `test` branch.

# Questions

Here are some questions that you might have come up with while reading this article:

*Why does git use a tree object? Couldn't we store the meta data stored by the tree object in the
commit object?*

One of the reasons that git uses tree objects is that they help in preserving the working directory
structure. Tree objects can store other tree objects which basically represent directories and
subdirectories in a working directory. Tree objects represent a snapshot of the working directory at
a point in time when a commit object is made.

Another reason is that different commits can have the same tree objects. If you followed along with
the commands in this guide verbatim, your git repo would have the same (the sha1 hashes would be
identical) blob and tree object as I did. However, your commit object sha1 hash will be different
than mine because of different meta data like the name and email of the author and committer. In
case git encounters the same tree object in different commits in a single repository, it won't
create new blob and tree objects and reuse the ones we already have which saves space and increases
efficiency.

*Why does git use the first two characters of the 40-digit hash of git objects as directory names and
the remaining 38-digits as the file name of the object itself?*

The layout of subdirectories in `.git/objects/` is in the form of `[0-9a-f][0-9a-f]/[0-9a-f]+` which
means that there can be 256 subdirectories in `.git/objects/`. Some file systems like FAT32 can have
only 65,536 files in a directory but, technically, there can be 2^60 SHA1 hashes. EXT2 has
performance issues post 10k files in a directory. EXT4 doesn't any such limits but it's still
limited by the number of inodes it can have.

In essence, this directory structure is the result of optimization keeping in mind the performance
limitations of file systems.

# References

Here are some resources that might help you get started with and understand git. This includes
comprehensive references as well. References specific to a certain topic will be included elsewhere.

- [Think Like a Git](http://think-like-a-git.net/)
- [Git From The Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
- [Git From The Inside Out](https://codewords.recurse.com/issues/two/git-from-the-inside-out)
- [Chapter 10 of the Pro Git Book](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

Here are some videos that use git internals to explain how git works.[^1]

- [Git From the Bits Up](https://www.youtube.com/watch?v=MYP56QJpDr4)
- [Dissecting Git Guts](https://www.youtube.com/watch?v=Y2Msq90ZknI)
- [Advanced Git: Graphs, Hashes, and Compression, Oh My!](https://www.youtube.com/watch?v=ig5E8CcdM9g)

Visualizing a git repository and its graph can become easier using:

- [this](https://git-school.github.io/visualizing-git/#free-remote) tool
- [this](https://learngitbranching.js.org/) tool which can help you visualize
  various operations on a git repository
- [gitDAGs](https://github.com/jubobs/gitdags/) LaTeX package might come in
  handy to draw git graphs. There are instructions on how to install it
  [here](https://chrisfreeman.github.io/gitdags_install.html). Yeah, I know,
  LaTeX. But unfortunately, I haven't found anything that looks better than
  this yet. Mermaid support for git graphs is experimental and looks shabby as
  of Dec 2020. Please let me know if you find something better.
- you can use software like [gitg](https://gitlab.gnome.org/GNOME/gitg) and the
  [gitgraph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)
  vscode extension to view pretty graphs of your repos.

These resources might help you as well. I haven't completed studying them myself though.

- [Git Internals by Scott Chacon](https://github.com/pluralsight/git-internals-pdf/)
- [A Visual Guide to Git Internals](https://www.freecodecamp.org/news/git-internals-objects-branches-create-repo/)
- [The chapter on Git in the AOSA book](https://www.aosabook.org/en/git.html)
- [Git Magic](http://www-cs-students.stanford.edu/~blynn/gitmagic/ch01.html)
- [Git From the Inside Out](https://maryrosecook.com/blog/post/git-from-the-inside-out)
- [Chapter 4. Basic Git Concepts](https://www.oreilly.com/library/view/version-control-with/9781449345037/ch04.html)

[^1]: If you like these videos, consider saving them off-line. In my opinion, YouTube cannot
be relied upon for data availability.
