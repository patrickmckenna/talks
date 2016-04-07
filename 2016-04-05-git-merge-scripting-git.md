template: title

.mega-octicon.octicon-circuit-board[]

# Scripting Git

Patrick McKenna

---

template: content

# Why? .octicon.octicon-circuit-board[]

Git contains tons of useful data and offers powerful commands...

--

...but the interface for those commands is often cumbersome to use

--

... and accessing Git data can be tricky

---

template: content

# What we'll cover .octicon.octicon-circuit-board[]

First, we'll give an introduction to writing shell scripts for Git

--

We'll develop 2 examples:

- List the new commits on a branch after a `git pull`

- Run a test against a range of commits

---

template: content

# What we'll cover .octicon.octicon-circuit-board[]

Next, we'll show how to interact directly with Git objects, using modern languages (Python, Ruby, Go, ...), thanks to libgit2

--

We'll develop an example using Rugged, the Ruby binding to libgit2:

- Git-backed web server

---

template: content

# Workshop mechanics .octicon.octicon-circuit-board[]

We'll do this hands-on, going slowly enough for everyone to write these scripts

--

Our assumptions:

- You're comfortable with command line Git, and understand the basics of its internals

- You familiar with the basics of shell scripting

- You have a local Git repo to mess about with

---

template: section

.mega-octicon.octicon-circuit-board[]

# Shell scripting and Git

---

template: content

# Simplify common tasks .octicon.octicon-circuit-board[]

We often ask the same questions of Git

--

For example: what commits were added to `master` by my last `pull`?

--

Let's answer this, but for an arbitrary branch (not just `master`)

```bash
git-show-new <branch>
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

First, we need accept a `<branch>` arg

If none is received, error out

--

```bash
#!/bin/bash

if [ $# -ge 1 ]; then
  branch="$1"
else
  echo "git-show-new requires a branch name!"
  exit 1
fi
# ...
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

Note:

- We explicitly invoke Bash

- We ignore anything past the first arg

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

Next, we need to generate the list of new commits

--

We use `git rev-list`

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

`git rev-list` fundamentally important plumbing command

--

It generates and traverses commit graphs

*Reminder: commits form a directed acyclic graph (DAG)*

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

How do we use it?

--

`git rev-list A ^B`

Lists commits reachable from `A` but not `B`

--

`A`, `B` anything that resolves to a commit (i.e. they're [commit-ish](https://www.kernel.org/pub/software/scm/git/docs/gitglossary.html))

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

`git rev-list A ^B`

In our case (assuming we care about `master`):

- `A` is the commit where `master` currently points

- `B` is the commit `master` pointed to before our latest `pull`

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

How do we specify the commit `master` used to point to?

--

Using Git's `@{...}` syntax

```bash
master@{1}
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

Hence

```bash
git rev-list master ^master@{1}
```

--

Equivalently, using alternate syntax (and generalizing to use our var `branch`)

```bash
git rev-list "$branch"@{1}.."$branch"
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

Should we output just a list of `SHA1`s?

--

Let's count them instead

```bash
git rev-list "$branch"@{1}.."$branch" | wc -l
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

We'll use `git log`, with same revision range syntax, to output commit info

```bash
git --no-pager log "$branch"@{1}.."$branch" --oneline
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

Putting it all together

```bash
#!/bin/bash

if [ $# -ge 1 ]; then
  branch="$1"
else
  echo "git-show-new requires a branch name!"
  exit 1
fi

printf "\n%s%s\n\n" $(git rev-list $branch@{1}..$branch | wc -l) \
  " commits were added by your last update to $branch:"
git --no-pager log "$branch"@{1}.."$branch" --oneline
```

---

template: content

# List commits added to `branch` by last `pull` .octicon.octicon-circuit-board[]

Notes:

--

- Requires an arg

--

- Doesn't accept multiple args

--

- Could also make this a Git alias

```bash
git config --global alias.show-new \
  "!f() { # contents of script go here } ; f"
```

---

template: section-with-subtitle

.mega-octicon.octicon-circuit-board[]

# Shell script example 2

test a range of commits

---

template: content

# A more in-depth example

Run a test on a range of commits, stopping (by default) if test fails

(based on @mhagger's [script](https://git.io/vVnfQ))

--

```bash
git-test-range [-k|--keep-going] RANGE -- COMMAND
```

---

template: content

# Testing a range of commits .octicon.octicon-circuit-board[]

Where do we start?

--

Need to verify:

- Script run from inside a valid repo

- Working directory is clean

---

template: content

# Get by with a little help from your Git-friends .octicon.octicon-circuit-board[]

--

Git includes `git-sh-setup` "scriplet"

Meant for inclusion in other scripts

- Performs some useful checks

- Offers helper functions

--

Let's use it!

---

template: content

# Using `git-sh-setup` .octicon.octicon-circuit-board[]

Make sure we run `git-test-range` inside valid repo?

--

Just source the script

```bash
. "$(git --exec-path)/git-sh-setup"
```

---

template: content

# Using `git-sh-setup` .octicon.octicon-circuit-board[]

What if we wanted to allow `git-test-range` to be run from anywhere?

--

Set var `NONGIT_OK` before sourcing `git-sh-setup`

```bash
# NONGIT_OK=true

# the source the script
. "$(git --exec-path)/git-sh-setup"
```

???

- `true`, `false` actually commands
- could also use non-empty string

---

template: content

# Using `git-sh-setup` .octicon.octicon-circuit-board[]

Check for clean working directory

--

```bash
# after sourcing script
require_clean_work_tree <command>
```

--

Pass `<command>` to add info to error message

---

template: content

# Testing a commit .octicon.octicon-circuit-board[]

Now define a function to actually test a commit

--

```bash
test_rev() {
    local rev="$1"              # keyword local a Bash thing
    local command="$2"
    git checkout -q "$rev" &&   # suppress feedback messages
        eval "$command"         # don't run command unless checkout successful
  # ...
}
```

---

template: content

# Testing a commit .octicon.octicon-circuit-board[]

Now define a function to actually test a commit

```bash
test_rev() {
    local rev="$1"
    local command="$2"
    git checkout -q "$rev" &&
        eval "$command"
    local retcode=$?              # shell functions can't return values, so we
    if [ $retcode -ne 0 ]         #   use return (exit) codes instead
    then
        printf "\n%s\n" "$command FAILED ON:"
        git --no-pager log -1 --decorate $rev
        return $retcode           # make test_rev's return code same as
    fi                            #   command's
}
```

---

template: content

# Store current revision .octicon.octicon-circuit-board[]

--

Need this so we can `checkout` back to commit we were on when we run `git-test-range`

--

```bash
head=$(git symbolic-ref HEAD 2>/dev/null || git rev-parse HEAD)
```

--

First command will error out if we run the script from a detached `HEAD` state

---

template: content

# Event loop .octicon.octicon-circuit-board[]

Define what we'll loop through

--

We already know about:

- `git rev-list`

- specifying commit ranges, e.g. `feature..master`

--

What's left? Dealing with test results...

---

template: content

# Event loop .octicon.octicon-circuit-board[]

```shell
fail_count=0
for rev in $(git rev-list --reverse $range); do
    test_rev $rev "$command"
    retcode=$?
    if [ $retcode -eq 0 ]; then               # all good, test next commit!
        continue
    fi
    # ...
```

---

template: content

# Event loop .octicon.octicon-circuit-board[]

```bash
fail_count=0
for rev in $(git rev-list --reverse $range); do
    test_rev $rev "$command"
    retcode=$?
    if [ $retcode -eq 0 ]; then
        continue
    fi
    if [ $keep_going ]; then                  # if a test fails, only continue if
        fail_count=$((fail_count + 1))        #   user chose that option
        continue
    else
        git checkout -fq ${head#refs/heads/}  # get back to where we started
        exit $retcode                         #   otherwise HEAD detached
    fi
done
git checkout -fq ${head#refs/heads/}          # get back to where we started
```

---

template: content

# Missing pieces .octicon.octicon-circuit-board[]

We've skipped a few things...

--

- Dealing with input args

--

- Printing out final results

--

Complete version here: `https://git.io/vVgTY`

---

template: section

.mega-octicon.octicon-circuit-board[]

# Limitations of scripting

---

template: content

# Limitations of scripting .octicon.octicon-circuit-board[]

Problems arise when you want to:

--

- Start doing complicated things

--

- Use a more modern, fully-featured language (often for above reason)

--

- Scale &#8212; shelling out to Git can get expensive

---

template: section

.mega-octicon.octicon-circuit-board[]

# libgit2 + bindings

---

template: content

# libgit2 .octicon.octicon-circuit-board[]

--

- Portable, pure C implementation of Git core methods

  - Git is mostly C, but some shell, Perl

--

- [Bindings to it](https://github.com/libgit2/libgit2#language-bindings) in most major languages

  - Python, Ruby, Node.js, Go, ...

--

- Well-established, actively maintained OSS project with wide industry support

  - GitHub, Microsoft, Atlassian, Canonical, others use in production

---

template: section-with-subtitle

.mega-octicon.octicon-circuit-board[]

# Rugged
Ruby binding to libgit2

---

template: content

# Why Rugged? .octicon.octicon-circuit-board[]

--

- Ruby benefits

  - Widely used

  - Very readable syntax (even for newcomers)

  - Great for scripting

  - Huge package ecosystem

--

- (Also: time limits)

---

template: content

# Installing Rugged .octicon.octicon-circuit-board[]

Rugged is distributed as a self-contained gem, so:

```shell
gem install rugged
```

It's easiest to let Rugged use its bundled version of libgit2; to do that, you'll need:

- CMake

- `pkg-config`

???

consider taking break here, so people can install as needed

---

template: content

# Rugged basics .octicon.octicon-circuit-board[]

Main object class (rarely created directly)

```ruby
Rugged::Object
```

--

Primary interface to local Git repos

```ruby
Rugged::Repository
```

---

template: content

# Rugged basics .octicon.octicon-circuit-board[]

Git has 4 fundamental object types

--

Rugged has corresponding class for each:

- `Rugged::Blob`

- `Rugged::Tree`

- `Rugged::Commit`

- `Rugged::Tag`

---

template: content

# We have Rugged, now what? .octicon.octicon-circuit-board[]

--

We can live in Ruby, use all of our favorite gems...

--

... *and* integrate Git natively!

---

template: content

# Making a Ruby app that uses Git .octicon.octicon-circuit-board[]

Ruby is widely used for building web apps

--

Let's build an app that serves a website directly from a Git branch

(Based on @carlosmn's [`git-httpd`](https://github.com/carlosmn/git-httpd))

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

--

Serve content directly from the Git object store (`.git` directory)

--

No need to checkout files onto disk

--

Example use case: locally deploy one branch that needs a web server (e.g. `gh-pages`), while you work on another

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

Let's make use of two Ruby gems

```shell
gem install sinatra
gem install mime
```

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

Starting our script

--

```ruby
#!/usr/bin/env ruby

require 'sinatra'
require 'rugged'
require 'mime/types'
```

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

Specify the repo, the branch(ref), and create the repo object

--

```ruby
repo_path = ENV['HOME'] + '/PATH/TO/REPO'
ref_name = 'refs/remotes/REMOTE/BRANCH'

repo = Rugged::Repository.new(repo_path)
```

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

Use Sinatra to start defining how our server responds to `GET` requests

--

```ruby
get '*' do |path|                         # Sinatra method
  commit = repo.ref(ref_name).target
  path.slice!(0)                          # strip leading slash
  path = 'index.html' if path.empty?
  # ...

```

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

Use the supplied path to retrieve the associated Git tree entry

--

```ruby
  # ...
  entry = commit.tree.path path   # entry is a Ruby hash (i.e. map/dictionary)
  puts path
  blob = repo.lookup entry[:oid]
  content = blob.content          # return contents of blob as string
  halt 404, "404 Not Found" unless content
  # ...
```

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

The whole thing (nearly)

--

```ruby
get '*' do |path|
  commit = repo.ref(ref_name).target
  path.slice!(0)
  path = 'index.html' if path.empty?

  entry = commit.tree.path path
  puts path
  blob = repo.lookup entry[:oid]
  content = blob.content
  halt 404, "404 Not Found" unless content

  content_type MIME::Types.type_for(path).first.content_type
  content
end
```

---

template: content

# Git-backed web server .octicon.octicon-circuit-board[]

Full example here: `https://git.io/vVgJp`

---

template: title

.mega-octicon.octicon-circuit-board[]

# Thank You!

Patrick McKenna

`patrickmckenna@github.com`

---

template: title

.mega-octicon.octicon-circuit-board[]

# Extras

---

template: content

# Understanding Git in modern terms .octicon.octicon-circuit-board[]

Git comes from the UNIX tradition

- Small programs designed to do one thing well

- Known, stable interfaces
  - `git rm`, `git mv`, ...

Sound like microservices...

---

template: content

# Git: just another microservices-oriented architecture .octicon.octicon-circuit-board[]

Git comes from the UNIX tradition

- Small programs designed to do one thing well

- Known, stable interfaces
  - `git rm`, `git mv`, ...

["We do not claim that the microservice style is novel or innovative, its roots go back at least to the design principles of Unix."](http://martinfowler.com/articles/microservices.html)
*Martin Fowler*

---

template: content

# Git: just another microservices-oriented architecture .octicon.octicon-circuit-board[]

Troll alert: let's not take that comparison too seriously

But, we can think writing a Git script as (borrowing Fowler's words again):

["developing a single application as a suite of small services, each running in its own [shell] process and communicating with lightweight mechanisms [e.g. `stdout`]"](http://martinfowler.com/articles/microservices.html)

---
