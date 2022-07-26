---
title: "Git LFS"
date: 2019-01-01
lastmod: 2019-03-17
draft: false
garden_tags: ["tech", "git"]
summary: "what problems does it solve, and what does it cost?"
status: "growing"
---

# Introduction - what is Git LFS?

Version control systems are great. Having a history which you can timetravel with is incredibly powerful.

Usually with VCS like Git or Mercurial, we when we write code we are making fairly lightweight text files. What if we want to use version history with much larger files - perhaps design assets like complex 3D videogame maps, or hi-res Photoshop drawings?

It's not wise to store large files like this in Git, as Git repositories store the full history: cloning a repo with 10 of our 1GB maps will be slow, as 10GB needs to download. And maybe you only need one (or even none!) of them straightaway.

... Enter Git Large File System (LFS). It allows you to save full files in remote storage, and only keep lightweight pointers to those files in your repository. Cloning the repository is fast, and you can download the large files when you need them. 

# For more general software engineers, what value could it add?

> "But I'm not a game developer, how can this be useful to me?"

Okay, here's the scenario I had in mind when exploring Git LFS: build systems like [Gradle](https://blog.gradle.org/introducing-incremental-build-support), Bazel, and mill can become much more efficient by only performing tasks when there's a change. If your source code hasn't changed, the compilation output won't change - so why compile anything? This avoids pointless work, saving time and money.

GitHub Actions seems to be becoming very popular now, and in theory it should integrate nicely with Git LFS [^1]. Whilst some Actions can use a cache, cache is not supported for Github Enterprise >3.5. If you prefer to work smarter not harder - why make infrastructure for an S3 bucket/policies/networking if you can use easier tools? So, I wondered:

[^1]: Here's a [StackOverflow](https://stackoverflow.com/a/61466160/4261132) post with example usage:

    ```shell
    steps:
      - name: Checkout github repo (+ download lfs dependencies)
        uses: actions/checkout@v3
        with:
          lfs: true
      - name: Checkout LFS objects
        run: git lfs checkout
    ````

    For checking out the latest version of a file on a branch, it would be like `git checkout origin/build-cache out.tar.gz`

> "Can I store build compilation output in Git LFS, i.e. as simple remote build cache"

# What does interacting with Git LFS look like?

Let's say we want to push a compilation output archive (.tar.gz) on a branch `build-cache`, to hide it from `main` branch. I borrowed this idea from Github Pages being able to find static website compilations (such as from Hugo) in the `gh-pages` branch.

```bash
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

```bash
git checkout -b aNewBranch

git lfs fetch origin build-cache

git checkout origin/build-cache output.tar.gz

// check we have the actual archive, not just a pointer
file output.tar.gz
```

For your curiosity, here's how the pointer looks:

```bash
version https://git-lfs.github.com/spec/v1
oid sha256:c37fecb501e8ce....
size 173
lfs-test.tar.gz (END)
```

Fortunately, I tested this with a *tiny* archive. More on that soon.

# Sounds pretty nice - what are the downsides of build cache with Git LFS?

This is where the bad news comes...

It's not easy to keep costs/remote storage lean with Git LFS.

> To remove Git LFS objects from a repository, delete and recreate the repository. [~ Github](https://docs.github.com/en/repositories/working-with-files/managing-large-files/removing-files-from-git-large-file-storage#git-lfs-objects-in-your-repository)

GitHub argues they are all about keeping history, not deleting it.
Git LFS says similar. On the other hand, to clear LFS remote on GitHub you have to delete all history, so it's up for debate really.

Depsite a lot of [desire for a smarter Git LFS for about 6 years](https://github.com/git-lfs/git-lfs/issues/1101), it doesn't look like it'll be coming around any time soon. There's lots of great suggestions/requests, like "last-only" backup, and rolling backup. I wonder who is using Git LFS, and how much it costs them... 


## What is the pricing models?

### GitHub: 
- [Free tier](https://docs.github.com/en/billing/managing-billing-for-git-large-file-storage/about-billing-for-git-large-file-storage#about-billing-for-git-large-file-storage): 1GB storage + 1GB bandwidth free per month
- [Paid tiers](https://docs.github.com/en/billing/managing-billing-for-git-large-file-storage/about-billing-for-git-large-file-storage#purchasing-additional-storage-and-bandwidth): $5 per month per 50GB storage + bandwidth
- **Example**: 1GB upload per day, 0.5GB download. First month cost: $5. Cost at 12 months: $35 per month.

That's actually not too bad. 
Using good compression to save space and bandwidth is sound advice. Bear in mind there's a tradeoff between using the best compression formats and algorithms VS using what people will already have (e.g. .tar.gz is decent for macos/ubuntu)
The problem comes with paying for stuff you don't need.

Also, be careful with bandwidth - if your quota runs out, ["Git LFS suport is disabled.... until the next month"](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-storage-and-bandwidth-usage#tracking-storage-and-bandwidth-use). I guess if you do a pull/push mechanism for build cache, you'll use 26GB storage but 52GB bandwidth. To reduce toil, I can foresee companies buying "a few" extra data packs.

It doesn't look like you can use a custom remote. GitHub manages it all for you, which could be a pro or a con.

### BitBucket: 
- [Various free tiers](https://support.atlassian.com/bitbucket-cloud/docs/storage-policy-for-git-lfs-with-bitbucket/#How-much-storage-do-I-get-with-my-pricing-plan), 1 or 5 GB depending on use-case
- [Paid tier](https://support.atlassian.com/bitbucket-cloud/docs/storage-policy-for-git-lfs-with-bitbucket/#How-much-does-additional-storage-for-Git-LFS-cost): $10 per 100GB file storage, no mention of bandwidth
- [10GB file size limit](https://support.atlassian.com/bitbucket-cloud/docs/manage-large-files-with-git-large-file-storage-lfs/) - probably plenty, but worth calling out (seen as they do!)

Unlike GitHub, BitBucket has a *Git LFS* view in a settings page where [LFS remote objects can be deleted](https://support.atlassian.com/bitbucket-cloud/docs/use-git-lfs-with-existing-bitbucket-repositories/#Delete-Git-LFS-files-from-a-repository). Better than nothing, but there's toil to save these costs (and probably not worth the time for a human to do it!). As you may have guessed, [someone has kindly automated this](https://gist.github.com/danielgindi/db0e0a897d8d920f23e155bb5d59e9c6) by running JS in Chrome's console. 

### GitLab: 
- You can setup custom object storage [via Fog](https://docs.gitlab.com/ee/administration/lfs/#storing-lfs-objects-in-remote-object-storage), e.g. on a private local network, or on aws
- For deleting remote files within Gitlab's set of tooling/services, it doesn't look like it's an option. GitLab seem to give incorrect advice that clearing the repo should also clear the remote ([1](https://stackoverflow.com/a/34582910/4261132), [2](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/30639)). Someone wrote a [custom Ruby hook](https://github.com/darshannnn/GitLabLFS-custom_hooks) to do this.

If you're already set up with AWS, you might be interested in estimating pricing. AWS provide a [calculator](https://calculator.s3.amazonaws.com/index.html) for that.


# Conclusion

If you *must* keep your large files versioned with Git, you can keep your repository lean with Git LFS. You just need to be aware of how costs will scale with time, and that with some Git providers you don't have much control over this.
# Extra notes

- GitLab - [getting started with Git LFS](https://about.gitlab.com/blog/2017/01/30/getting-started-with-git-lfs-tutorial/). There's a nice little advert for Tower showing integration with Git LFS in your usual Windows/Mac file browser. That could be nice for digital artists.
