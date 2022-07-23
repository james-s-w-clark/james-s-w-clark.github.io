---
title: "Git LFS"
date: 2019-01-01
lastmod: 2019-03-17
draft: false
garden_tags: ["tech"]
summary: "what problems does it solve, and what does it cost?"
status: "growing"
---

# tl;dr
Git LFS gives you versioning of large files, at an ever-growing cost.

# Introduction - what is Git LFS?

Version control systems are great. Having a history which you can timetravel with is incredibly powerful.

Usually with VCS like Git or Mercurial, we when we write code we are making fairly lightweight text files. What if we want to use version history with much larger files - like complex 3D videogame maps 1GB in size?

It's not wise to store large files like this in Git, as Git repositories store the full history: cloning a repo with 10 of our 1GB maps will be slow, as 10GB needs to download. But you only want one of them, and you might not need them yet...

... Enter Git Large File System (LFS). It allows you to save full files in remote storage, and only keep lightweight pointers to those files in your repository. Cloning the repository is fast, and you can download the large files when you need them.

```
version https://git-lfs.github.com/spec/v1
oid sha256:c37fecb501e8ce025581e77fbe5a0e7277fa9a745d29ce7a9cad6034778892ac
size 173
lfs-test.tar.gz (END)
```

# For more general software engineers, what value could it add?

> "But I'm not a game developer, how can this be useful to me?"

Okay, here's the scenario I had in mind when exploring Git LFS: build systems like [Gradle](https://blog.gradle.org/introducing-incremental-build-support), Bazel, and mill can become much more efficient by only performing tasks when there's a change. If your source code hasn't changed, the compilation output won't change - so the output can be reused. Avoiding pointless work saves time and money.

GitHub Actions seems to be becoming very popular now, and in theory it should integrate nicely with Git LFS. Whilst some Actions can use a cache, cache is not supported for Github Enterprise >3.5. I'm also lazy, and would prefer work smarter not harder - why make infrastructure for an S3 bucket if I can use easier tools? So, I wondered:

> "Can I store build compilation output in Git LFS, i.e. a simple remote build cache"

# What does interacting with Git LFS look like?

Let's say we want to push a compilation output archive (.tar.gz) on a branch `build-cache`, to hide it from `main` branch. I borrowed this idea from Github Pages being able to find static website compilations (such as from Hugo) in the `gh-pages` branch.

```
// https://git-lfs.github.com/
brew install git-lfs
// install various git hooks for nice git lfs syncing
git lfs install

// in a repo
git lfs track "*.tar.gz"

// git attributes is now tracking this filetype with git lfs
git add .gitattributes
git add output.tar.gz

git lfs push -u origin build-cache --all
```

Now we have our archive 1. in git history, 2. in a remote object store. Let's go to another branch

```
git checkout -b aNewBranch

git lfs fetch origin build-cache

git checkout origin/build-cache output.tar.gz

// check we have the actual archive, not just a pointer
file output.tar.gz
```

Fortunately, I tested this with a *tiny* archive. More on that soon.

# Sounds pretty nice - what are the downsides of build cache with Git LFS?

This is where the bad news comes...


GitHub argues they are all about keeping history, not deleting it.
Git LFS says similar.

Depsite a lot of desire for a rolling backup with Git LFS for about 6 years, it doesn't look like it'll be coming around any time soon.

# Closing notes

