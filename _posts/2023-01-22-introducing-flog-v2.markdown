---
layout: post
title:  "Introducing Flog v2, the first real Git branch viewer for Vim"
date:   2023-01-22 15:26:26 -0500
---
Today, I'm releasing [Flog v2](https://github.com/rbong/vim-flog), a Git branch viewer plugin for Vim.

Flog v2 is the first *real* Git branch viewer for Vim. I want to share what that means, why that's good, and how real branch viewers work.

## Why "real"?

Other branch viewer plugins exist - not to mention Flog v1. So why am I calling it the first "real" branch viewer?

Every other Git branch viewer plugin for Vim can't draw Git branches for itself. Under the hood, they all call the shell command `git log --graph`, which they use to draw branches for them.

Usually, this command is meant to be called in the terminal.

Here's how Flog v1, which uses `git log --graph`, looks:

![Screenshot of Flog v1, which uses "git log --graph"](/img/2023-01-22-flog-v1.png)

Here's how Flog v2, which draws branches itself, looks:

![Screenshot of Flog v2 with custom output](/img/2023-01-22-flog-v2.png)

But other than using pretty Unicode characters, what's the big deal? Why is this a better way to build a branch viewer? A few reasons.

## Fake branch viewers are inflexible

The first big problem we have when relying on `git log --graph` is that we don't know where particular branches are located in the output. This limits our ability to do anything related to visual navigation.

We also have to stick to an output format we know we can parse unless we start getting hacky. This brings us to our next problem - speed.

## Fake branch viewers are slow

We might want to add some flexibility to our branch viewer such as custom output formats and multiline output formats.

To do this:

* We can instruct `git log --graph` to add a bunch of extra tokens into the output.
* We can parse the output and get the data we need using the tokens.
* We can rebuild the output as it should appear to the user.

This isn't intended, but it works.

If we draw branches ourselves, we can skip all these steps. We don't need tokens to find the data we need because there aren't ASCII branches in the way.

It turns out that doing it for ourselves is much faster. To bring us to our next point - it also looks better.

## Fake branch viewers are ugly

The output of `git log --graph` is meant to be displayed in a terminal. It uses a predefined set of ASCII characters. This is not always pretty.

In a fake branch viewer, there's very little we can do about this. If we build the output ourselves, we can use any Unicode characters we want.

On top of this, `git log --graph` branches move to the left and right. Vim is not built to highlight shifting vertical columns.

When we draw the output ourselves, we can draw branches that stay straight and are easy to highlight.

## How to build a real branch viewer

Now you know about the downsides of "fake" branch viewers, so how do you build a real one? (If you just want to know about Flog, you can [skip this section](#what-else-can-flog-do).)

First, we shouldn't use `git log --graph`, but we can still use `git log`. This output is much easier to parse because it doesn't have a bunch of branch characters included in the output.

![Example of "git log"](/img/2023-01-22-flog-basic-example.png)

Next, we need to make sure `git log` sorts commits the same way it would when using `--graph`. It would sort these commits "topographically", which places them in an order that makes the branch structure easier to understand.

To do this, we can use the command `git log --topo-order`.

![Example of "git log --topo-order"](/img/2023-01-22-flog-topo-example.png)

Next, we need to know what commit has which parents. We can add this information to the output using `git log --format='%h %p'`.

The `%h` format specifier shows the commit hash, and the `%p` format specifier shows the parent hashes.

We also need to pass the `--parents` flag to `git log` to make sure it always includes parents in the output. Without this flag, Git will sometimes exclude parents.

![Example of "git log --parents --topo-order --parents --format='%h %p'"](/img/2023-01-22-flog-parents-example.png)

Now we have all the information we need. We can use this information to draw the output.

When we encounter new parents, we can add new branches to the output. When we encounter the parent commit, we can place it on its branch.

![Example of drawing branches](/img/2023-01-22-flog-branches-example.png)

After each commit, we must check if each branch is a parent of the current commit. If it is, we need to draw it connected to the current branch.

![Example of drawing merges](/img/2023-01-22-flog-merges-example.png)

There are many different possible ways for branches to fork off each other and merge together. For example, an octopus merge can merge several commits at the same time.

![Example of drawing octopus merges](/img/2023-01-22-flog-octopus-example.png)

The best way to check that different possible merges all appear correctly is by adding tests. See the [Flog test output](https://github.com/rbong/vim-flog/tree/v2.0.0/t/data) for some ideas of what to test.

After we're sure that our branch viewer can handle drawing all different types of connections, we're done handling drawing branches.

To summarize, we can draw branches using the following basic algorithm:

1. Parse the current commit and its parent commits.
2. Draw the current branches and the current commit.
3. If there are commits without branches, place new branches for them.
4. If there are parent branches, draw them connected to the current branch.

Next, if we simply record information about which commits go where while we're drawing our output, we should be able to use this information to implement navigation commands for our branch viewer.

Finally, it's a good idea to build a "real" branch viewer using a "real" language. I wouldn't recommend using VimScript to implement your entire branch viewer.

Flog v2 uses [Lua](https://www.lua.org/). Lua is much faster than VimScript.

## What else can Flog do?

Flog has navigation bindings that let you jump between commits, jump between branches, modify the output, and more.

It has extensive context-sensitive autocompletion which can run commands against the commit under the cursor or in the current visual range. After running commands, Flog updates automatically.

Flog also allows you to open commit information in a split next to the branch view.

![Opening a commit in Flog v2](/img/2023-01-22-flog-view-commit.png)

Flog has lots of cool features, but the coolest might be that if you open Flog up with a visual range selected, you can view the history of that range, complete with optional inline diff.

The diff even has syntax highlighting.

![Viewing an inline diff in Flog v2](/img/2023-01-22-flog-inline-diff.png)

Flog fully integrates with [Fugitive](https://github.com/tpope/vim-fugitive), a great Git plugin for Vim. Flog comes with lots of bindings that should be familiar to Fugitive users.

If all that is not enough, Flog is highly customizable and extensible.

If you'd like to try Flog, [get started here](https://github.com/rbong/vim-flog).

*Thanks for reading!*
