template: title

.mega-octicon.octicon-git-merge[]

# git-imerge

Patrick McKenna

---

template: section

.mega-octicon.octicon-git-merge[]

# Overview

---

template: content

# Overview .octicon.octicon-git-merge[]

- Why you need it

- What it does

- How to use it

---

template: content

# Combining history .octicon.octicon-git-merge[]

Two standard ways to combine branches:

- `git merge`

- `git rebase`

--

Both have significant limitations

---

template: content

# `git merge` limitations .octicon.octicon-git-merge[]

- Forces you to resolve *one big conflict* composed of many smaller changes

--

- Is *all-or-nothing* &#8212; there is *no way to save* a partially completed merge

--

  - You can't record your progress
  - You can't switch to another branch temporarily
  - If you make a mistake, you can't go back
  - If you can't resolve the *whole* conflict, you have to start over

---

template: content

# `git merge` limitations .octicon.octicon-git-merge[]

- There is *no way to test* a partially completed merge

  - Code typically won't even build/run until the conflict is completely resolved

--

- It is *difficult to collaborate* with others to resolve conflicts

---

template: content

# `git rebase` limitations .octicon.octicon-git-merge[]

- Forces you to reconcile each of the commits on `branch` with *all* of the changes on `master`

--

- Often requires similar conflicts to be *resolved multiple times*

--

- It is best done as an *all-or-nothing* operation &#8212; awkward to interrupt a rebase in progress

---

template: content

# `git rebase` limitations .octicon.octicon-git-merge[]

- It *discards history*

--

- Rewriting published history is *unfriendly to collaboration*

---

template: section-with-subtitle

.mega-octicon.octicon-rocket[]

# git-imerge

...to the rescue

---

template: content

# `git-imerge` overview .octicon.octicon-git-merge[]

Implements a new merging method, *incremental merge*, that reduces or eliminates the problems listed earlier

--

It considers the smallest possible merges, taking *one commit from each branch at a time*

--

This lets you tackle the smallest possible conflicts

---

template: content

# `git-imerge` overview .octicon.octicon-git-merge[]

`git-imerge` records all these intermediate merges

--

When finished, it lets you permanently store those, or discard them

--

You decide how to store the final result:

--

  - As a typical 3-way merge

  - As a rebase

  - As a rebase-with-history

---

template: content

# `git-imerge` advantages .octicon.octicon-git-merge[]

--

- Can be interrupted

--

- Can save the state of a partially complete merge

--

- Lets you push that state to a server

--

- Enables a colleague to pull that state, work on it further, and push their results

---

template: content

# `git-imerge` advantages .octicon.octicon-git-merge[]

- Lets you test those intermediate states (i.e. partially completed merges)

--

- Enables you to redo problematic merges if you make a mistake

--

- Never shows the same conflict twice

---

template: content

# `git-imerge` advantages .octicon.octicon-git-merge[]

`git-imerge` has very few downsides

--

- Most of those incremental merges are done automatically

--

- It's very efficient

---

template: section

.mega-octicon.octicon-git-merge[]

# What does all this look like?

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

Suppose you want to merge `branch` into `master`

```shell
o - 0 - 1 - 2 - 3 - 4 - 5 - 6 - 7 - 8 - 9 - 10 - 11    ← master
     \
      A - B - C - D - E - F - G                        ← feature

```

--

First draw the diagram a bit differently...

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |
    A
    |
    B
    |
    C
    |
    D
    |
    E

    ↑
  feature
```

--

Now start filling it in, merging commits pairwise...

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
    |
    B
    |
    C
    |
    D
    |
    E

    ↑
  feature
```

---

template: content

# `git-imerge` conceptually .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
```

--

`A1` is a merge commit between commit `1` on `master` and commit `A` on `feature`

--

It has two parents, like any merge commit

--

`git-imerge` stores commit `A1` to your repository

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
    |   |
    B - B1
    |
    C
    |
    D
    |
    E

    ↑
  feature
```

---

template: content

# `git-imerge` conceptually .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
    |   |
    B - B1
```

--

`B1` is a merge between `A1` and `B`

--

`B1` only has to add the changes from commit `B` to state `A1`

Equivalently, it adds the changes from `1` into state `B`

---

template: content

# `git-imerge` conceptually .octicon.octicon-git-merge[]

Most of these pairwise merges will not conflict; `git-imerge` will do them for you automatically

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
    |   |
    B - B1
    |   |
    C - C1
    |
    D
    |
    E

    ↑
  feature
```

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
    |   |
    B - B1
    |   |
    C - C1
    |   |
    D - D1
    |
    E

    ↑
  feature
```

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |
    A - A1
    |   |
    B - B1
    |   |
    C - C1
    |   |
    D - D1
    |   |
    E - E1

    ↑
  feature
```

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |
    B - B1
    |   |
    C - C1
    |   |
    D - D1
    |   |
    E - E1

    ↑
  feature
```

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |    |
    B - B1 - B2
    |   |
    C - C1
    |   |
    D - D1
    |   |
    E - E1

    ↑
  feature
```

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |    |
    B - B1 - B2
    |   |    |
    C - C1 - C2
    |   |
    D - D1
    |   |
    E - E1

    ↑
  feature
```

---

Falling asleep?

--

Quiz time!

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

Quick: what is the meaning of `C2`?

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |    |
    B - B1 - B2
    |   |    |
    C - C1 - C2
    |   |
    D - D1
    |   |
    E - E1

    ↑
  feature
```

---

template: content

# `git-imerge` conceptually .octicon.octicon-git-merge[]

Quick: what is the meaning of `C2`?

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |    |
    B - B1 - B2
    |   |    |
    C - C1 - C2
```

It adds the changes from commit `C` to state `B2`

Equivalently, it adds the changes from commit `2` to state `C1`

---

template: content

# Take-home message .octicon.octicon-git-merge[]

--

- Each merge adds the changes from *one single* commit to a state that has already been committed

--

- Git knows a nearby common ancestor

--

And so `git-merge` proceeds

--

Until...

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |    |
    B - B1 - B2
    |   |    |
    C - C1 - C2
    |   |    |
    D - D1 - D2
    |   |    |
    E - E1 - X

    ↑
  feature
```

--

It encounters a conflict

---

template: content

# `X` marks the spot

`git-imerge` needs your help

You have to manually merge `E1` and `D2` to make a commit `E2`

---

template: content

# `X` marks the spot

```shell
    D - D1 - D2
    |   |    |
    E - E1 - X
```

The commits share `D1` as a common ancestor (merge base)

You need to add the change made in commit `2` to state `E1`

Equivalently, add the change made in commit `E` to state `D2`

---

template: content

# Another take-home lesson! .octicon.octicon-git-merge[]

--

`git-imerge` is showing you smallest possible conflicts, *when they first appear*

--

In a typical `merge` or `rebase`, many such pairwise conflicts will pop up

--

Now you can deal with each of them in isolation!

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

Once resolved...

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |
    A - A1 - A2
    |   |    |
    B - B1 - B2
    |   |    |
    C - C1 - C2
    |   |    |
    D - D1 - D2
    |   |    |
    E - E1 - E2

    ↑
  feature
```

---

template: content

# `git-imerge` schematically .octicon.octicon-git-merge[]

... you continue until the merge diagram gets filled in

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |    |    |    |    |    |    |    |     |     |
    A - A1 - A2 - A3 - A4 - A5 - A6 - A7 - A8 - A9 - A10 - A11
    |   |    |    |    |    |    |    |    |    |     |     |
    B - B1 - B2 - B3 - B4 - B5 - B6 - B7 - B8 - B9 - B10 - B11
    |   |    |    |    |    |    |    |    |    |     |     |
    C - C1 - C2 - C3 - C4 - C5 - C6 - C7 - C8 - C9 - C10 - C11
    |   |    |    |    |    |    |    |    |    |     |     |
    D - D1 - D2 - D3 - D4 - D5 - D6 - D7 - D8 - D9 - D10 - D11
    |   |    |    |    |    |    |    |    |    |     |     |
    E - E1 - E2 - E3 - E4 - E5 - E6 - E7 - E8 - E9 - E10 - E11

    ↑
  feature
```

---

template: section

.mega-octicon.octicon-git-merge[]

# Understanding the results

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

What exactly are we looking at?

--

A completed incremental merge contains all the info you'd want to know about combining two branches

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

Where is the merge of `feature` and `master`?

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |    |    |    |    |    |    |    |     |     |
    A - A1 - A2 - A3 - A4 - A5 - A6 - A7 - A8 - A9 - A10 - A11
    |   |    |    |    |    |    |    |    |    |     |     |
    B - B1 - B2 - B3 - B4 - B5 - B6 - B7 - B8 - B9 - B10 - B11
    |   |    |    |    |    |    |    |    |    |     |     |
    C - C1 - C2 - C3 - C4 - C5 - C6 - C7 - C8 - C9 - C10 - C11
    |   |    |    |    |    |    |    |    |    |     |     |
    D - D1 - D2 - D3 - D4 - D5 - D6 - D7 - D8 - D9 - D10 - D11
    |   |    |    |    |    |    |    |    |    |     |     |
    E - E1 - E2 - E3 - E4 - E5 - E6 - E7 - E8 - E9 - E10 - E11

    ↑
  feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

`E11`

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11
    |                                                       |
    A                                                       |
    |                                                       |
    B                                                       |
    |                                                       |
    C                                                       |
    |                                                       |
    D                                                       |
    |                                                       |
    E ---------------------------------------------------- E11  ← master

    ↑
  feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

Usually such a diagram is drawn like this

```shell
o - 0 - 1 - 2 - 3 - 4 - 5 - 6 - 7 - 8 - 9 - 10 - 11 - E11    ← master
     \                                               /
      A -------- B -------- C --------- D --------- E        ← feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

Where is the rebase of `feature` onto `master`?

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |    |    |    |    |    |    |    |     |     |
    A - A1 - A2 - A3 - A4 - A5 - A6 - A7 - A8 - A9 - A10 - A11
    |   |    |    |    |    |    |    |    |    |     |     |
    B - B1 - B2 - B3 - B4 - B5 - B6 - B7 - B8 - B9 - B10 - B11
    |   |    |    |    |    |    |    |    |    |     |     |
    C - C1 - C2 - C3 - C4 - C5 - C6 - C7 - C8 - C9 - C10 - C11
    |   |    |    |    |    |    |    |    |    |     |     |
    D - D1 - D2 - D3 - D4 - D5 - D6 - D7 - D8 - D9 - D10 - D11
    |   |    |    |    |    |    |    |    |    |     |     |
    E - E1 - E2 - E3 - E4 - E5 - E6 - E7 - E8 - E9 - E10 - E11

    ↑
  feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

The right-most column

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
                                                            |
                                                           A11'
                                                            |
                                                           B11'
                                                            |
                                                           C11'
                                                            |
                                                           D11'
                                                            |
                                                           E11'  ← feature

```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

Usually that's drawn like this

```shell
0 - 1 - ... - 11    ← master
               \
                A' - B' - C' - D' - E'       ← feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

Where is the rebase of `master` onto `feature`?

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |    |    |    |    |    |    |    |     |     |
    A - A1 - A2 - A3 - A4 - A5 - A6 - A7 - A8 - A9 - A10 - A11
    |   |    |    |    |    |    |    |    |    |     |     |
    B - B1 - B2 - B3 - B4 - B5 - B6 - B7 - B8 - B9 - B10 - B11
    |   |    |    |    |    |    |    |    |    |     |     |
    C - C1 - C2 - C3 - C4 - C5 - C6 - C7 - C8 - C9 - C10 - C11
    |   |    |    |    |    |    |    |    |    |     |     |
    D - D1 - D2 - D3 - D4 - D5 - D6 - D7 - D8 - D9 - D10 - D11
    |   |    |    |    |    |    |    |    |    |     |     |
    E - E1 - E2 - E3 - E4 - E5 - E6 - E7 - E8 - E9 - E10 - E11

    ↑
  feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

The bottom-most row

```shell
o - 0
    |
    A
    |
    B
    |
    C
    |
    D
    |
    E - E1'- E2'- E3'- E4'- E5'- E6'- E7'- E8'- E9'- E10'- E11'  ← master

    ↑
  feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

Usually that `rebase` diagram is drawn like so

```shell
                  1' - 2' - 3' - 4' - 5' - 6' - 7' - 8' - 9' - 10' - 11'  ← master
                 /
      A - ... - E       ← feature
```

---

template: content

# Understanding the results .octicon.octicon-git-merge[]

`git-imerge` lets you keep the merge or the rebase

--

You can discard all of the intermediate commits, or keep them

--

You can also choose another option: "rebase with history"

---
template: content

# New and improved `rebase`, now with history .octicon.octicon-git-merge[]

If you have already published `feature`...

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |    |    |    |    |    |    |    |     |     |
    A - A1 - A2 - A3 - A4 - A5 - A6 - A7 - A8 - A9 - A10 - A11
    |   |    |    |    |    |    |    |    |    |     |     |
    B - B1 - B2 - B3 - B4 - B5 - B6 - B7 - B8 - B9 - B10 - B11
    |   |    |    |    |    |    |    |    |    |     |     |
    C - C1 - C2 - C3 - C4 - C5 - C6 - C7 - C8 - C9 - C10 - C11
    |   |    |    |    |    |    |    |    |    |     |     |
    D - D1 - D2 - D3 - D4 - D5 - D6 - D7 - D8 - D9 - D10 - D11
    |   |    |    |    |    |    |    |    |    |     |     |
    E - E1 - E2 - E3 - E4 - E5 - E6 - E7 - E8 - E9 - E10 - E11

    ↑
  feature
```

---

template: content

# New and improved `rebase`, now with history .octicon.octicon-git-merge[]

... consider using this mixed approach

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11   ← master
    |                                                       |
    A ---------------------------------------------------- A11'
    |                                                       |
    B ---------------------------------------------------- B11'
    |                                                       |
    C ---------------------------------------------------- C11'
    |                                                       |
    D ---------------------------------------------------- D11'
    |                                                       |
    E ---------------------------------------------------- E11'

                                                            ↑
                                                          feature
```

---

template: content

# New and improved `rebase`, now with history .octicon.octicon-git-merge[]

Rebase-with-history is a useful hybrid between rebasing and merging:

--

- It retains both the old and the new versions of `feature`

- It avoids the typical problems associated with rewriting published history

---

template: section

.mega-octicon.octicon-git-merge[]

# Using `git-imerge`

---

template: content

# Requirement, installation .octicon.octicon-git-merge[]

You'll need Python:

- Python 2.x, version 2.6 or later

- Python 3.x, version 3.3 or later

Repo itself: `github.com/mhagger/git-imerge`

OS X users: it's available via `homebrew`, too

---

template: section

.mega-octicon.octicon-git-merge[]

# Live coding!

---

template: content

# Using `git-imerge` .octicon.octicon-git-merge[]

Useful script to generate files w/ random contents:

`https://git.io/vV2rI`

---

template: title

.mega-octicon.octicon-git-merge[]

# Thank you!

`github.com/mhagger/git-imerge`

Patrick McKenna

`patrickmckenna@github.com`

---

template: section

.mega-octicon.octicon-git-merge[]

# Additional info

---

template: content

# Efficient implementation .octicon.octicon-git-merge[]

`git-imerge` does not usually have to complete every incremental merge

--

Its bisection-based algorithm fills in large blocks of the incremental merge, and quickly identify conflicts

---

template: content

# Efficient implementation .octicon.octicon-git-merge[]

For example, a typical in-progress merge might look like this

```shell
o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
    |   |    |    |    |    |    |    |    |    |     |     |
    A - A1 - A2 - A3 - A4 - A5 - A6 - A7 - A8 - A9 - A10 - A11
    |   |    |    |    |    |    |    |    |
    B - B1 - B2 - B3 - B4 - B5 - B6 - B7 - B8   X
    |   |    |    |    |    |    |
    C - C1 - C2 - C3 - C4 - C5 - C6   X
    |   |    |    |    |    |    |
    D - D1 - D2 - D3 - D4 - D5 - D6
    |   |    |    |    |    |    |
    E - E1 - E2 - E3 - E4 - E5 - E6

    ↑
  feature
```

(`X`s mark pairwise merges that conflict)

---

template: content

# Efficient implementation .octicon.octicon-git-merge[]

But `git-imerge` only needs to compute this<sup>1</sup>

```shell
  o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
      |   |    |    |    |    |    |    |    |    |     |     |
      A -   --   --   --   --   -- A6 -   -- A8 - A9 - A10 - A11
      |   |    |    |    |    |    |    |    |
      B -   --   --   --   --   -- B6 - B7 - B8   X
      |   |    |    |    |    |    |
      C -   --   --   --   --   -- C6   X
      |   |    |    |    |    |    |
      D -   --   --   --   --   -- D6
      |   |    |    |    |    |    |
      E - E1 - E2 - E3 - E4 - E5 - E6

      ↑
    feature
```

<sup>1</sup>plus a few test merges to locate the conflicts

---

template: content

# Implementation details: efficiency .octicon.octicon-git-merge[]

The gaps could be skipped because the merges on the boundaries were all successful

```shell
  o - 0 - 1  - 2  - 3  - 4  - 5  - 6  - 7  - 8  - 9  - 10  - 11    ← master
      |   |    |    |    |    |    |    |    |    |     |     |
      A -   --   --   --   --   -- A6 -   -- A8 - A9 - A10 - A11
      |   |    |    |    |    |    |    |    |
      B -   --   --   --   --   -- B6 - B7 - B8   X
      |   |    |    |    |    |    |
      C -   --   --   --   --   -- C6   X
      |   |    |    |    |    |    |
      D -   --   --   --   --   -- D6
      |   |    |    |    |    |    |
      E - E1 - E2 - E3 - E4 - E5 - E6

      ↑
    feature
```

---
template: content

# Intermediate data .octicon.octicon-git-merge[]

`git-imerge` store intermediate results as references under `refs/imerge/NAME/`

--

- `refs/imerge/NAME/state`
  - A blob containing a little bit of metadata

--
- `refs/imerge/NAME/{manual,auto}/M-N`
  - Manual/automatic merge that includes all of the changes through commits `M` on `master` and `N` on `feature`

--
- `refs/heads/imerge/NAME`
  - Temporary branch used when resolving merge conflicts

--
- `refs/heads/NAME`
  - Default branch where final results are stored
