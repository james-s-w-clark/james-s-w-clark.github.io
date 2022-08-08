---
title: "How to always know when there are PRs to check out 🔮"
date: 2022-08-05
lastmod: 2022-08-05
draft: false
garden_tags: ["tooling"]
summary: "... I can see everything 👀"
status: "seeding"
---

A few weeks ago I heard of SwiftBar, and [Xbar](https://xbarapp.com/):

| Put the output from any script or program into your macOS Menu Bar

There's a great library of plugins on the hub, covering all sorts of topics. Want to show the weather? Show upcoming info about your favourite sports team? Check if graphics cards are in stock? It can do all of this.

What's also great is the fact that you can see the source code of all the plugins. You can tweak those plugins and make them your own - and it's interesting to understand how people solved these kinds of problems.

# Playing with plugins 😳

I installed a few plugins, said "wow", and quickly realised that the Mac Pro's camera notch is a bit annoying - you lose a fair bit of real estate for tools like this! I used [Dozer](https://github.com/Mortennn/Dozer) / [Hidden](https://github.com/dwarvesf/hidden) to get rid of a few non-essential plugins, but they take a couple icons themselves so it didn't help much.

This taught me that I should be as lean as possible with how many characters I display with any plugin I write. Lots of plugins stick to emojis and numbers - I agree 💯%

# How an xbar plugin works

The [docs](https://github.com/matryer/xbar-plugins/blob/main/CONTRIBUTING.md) for xbar are pretty good. There's really two parts to a plugin: getting your data, and making xbar display it. 

To show your data, you just echo/println it. How the display is structured (icon, click-on-icon-modal, etc.) is like this:

```bash
#!/usr/bin/env bash
# Cycle through 
echo "cycle-one"
echo "cycle-two"
echo "cycle-three"
echo "---"
# Next lines only visible in dropdown for this plugin
echo "dropdown-item1"
echo "dropdown-item2"
echo "--dropdown-item2-something"
echo "----dropdown-item2-something-yodawg"
echo "--dropdown-item2-somethingElse"
# Next item is also in dropdown, separated by horizontal bar
echo "---"
echo "dropdown-item3"
```

{{< figure src="xbar_tree.jpg" width="100%" >}}

I think that's far more nesting that I'd ever want to use - but it's nice that it's there if you want it. I guess somebody did!

Anyway, you can put links/colours/shortcuts in there and make it clickable. You can do other stuff like have the user enter variables (<xbar.var>) for the program to access as environment variables (it can't access your actual environment variables).

Another key feature is deciding how often your plugin updates. The plugin filenames should have a format of {name}.{time}.{ext}, e.g. check_weather.15m.sh.

# Writing an xbar plugin

Lots of teams use Github PRs. xbar seemed like the perfect tool for me to monitor PRs. I've tried a few systems for monitoring PRs, but I'm not a fan of any of them:

- I don't like *email* spam (one more app/tab, multitasking) 
- *Chatbots* are either slow (quickly oudated) or spammy
- *Manually checking the site* either needs a few page loads or a bookmark - either way, it's not the smoothest experience... but the problem for me is it doesn't say in advance if there will be any information for you to act on

## Goals
 
I figured an xbar plugin would let me:
- know if I can to merge (required approval count is met, CI checks pass)
- know if I have any comments/suggestions to address
- get a count of how many PRs other people have open
- get a count of how many PRs other people need reviews on
- see if PR checks pass/fail
- do this over multiple repositories

Your ideology on PRs probably makes you question these goals... but you can tweak plugins, and you can tweak them to match your ideologies. 

# Writing this plugin with bash

The Github CLI is quite powerful - it can handle a lot of complexity with a clean API:

```bash
#!/bin/bash
username=$"james"
repo=$"ourOrg/ourBigRepo"

all_prs=$(gh pr list --repo $repo)

echo "$all_prs"

my_prs=$(gh pr list --repo WeaponX/ourBigRepo --author $username)
echo "$my_prs"

# TODO take to browser with --web
```

There's a lot of [options/args](https://cli.github.com/manual/gh_pr_list) you can use, and other handy commands like [gh pr status](https://cli.github.com/manual/gh_pr_status). The CLI can natively output to json or use jq filters.

I had a think about how I'd use this to tick off those goals, but for me it seemed easier to use a language I'm more familar with. 

- node.js
    - + I get higher order functions (map, filter, reduce, etc.) which can help me focus on what I want, not how to do it
    - - might need a global npm install of axios for making requests
    - ~~- need to figure out how to run a node file as a bash-like file ()~~
        - that only took a second to find out and verify - [it's pretty much what you'd expect](https://stackoverflow.com/a/24183402/4261132)
- scala
    - it's the language I work with, so it'd be good to practice more 
    - [scala-cli](https://scala-cli.virtuslab.org/docs/guides/scripts#self-executable-scala-script) can let you write self-executable scala scripts
        - I was curious how the experience would be, so I went with this


Of course, sticking with bash would be nice. The Github CLI could be used for simplicity, which some users may already have - or curl could be used for even fewer dependencies. I encourage anyone who finds that challenge interesting to give it a try!

# Writing this plugin with ~~bash~~ Scala

The first question is: Node has Axios, Python has Requests... but what's nice and simple for Scala? Enter Li Haoyi's [requests-scala](https://github.com/com-lihaoyi/requests-scala). Let's install [scala-cli](https://scala-cli.virtuslab.org/install) and have a try:

```bash
#!/usr/bin/env -S scala-cli shebang
import $ivy.`com.lihaoyi::requests:0.7.1`
import $ivy.`com.lihaoyi::ujson:2.0.0`

val resp = requests.get("https://api.github.com/users/lihaoyi")
val data = ujson.read(resp.text())
println(data("login")) # username
```

It's close, it just needs a little tweak to the suggested shebang: for my MacOs brew install, `#!/usr/bin/env /opt/homebrew/bin/scala-cli` works. Scala-cli is pulling in those Ivy dependencies and using them, very nice.

To keep the blog short, you can what I've got so far here: [xbar-github-pr](https://github.com/IdiosApps/xbar-github-pr). At this time it just shows PR counts summed over multiple repositories for your PRs and others'.

TODO - add a screenshot... or maybe in the goals section to get interest earlier?

For now it's basic but it's a useful gauge/reminder. It can definitely be more useful, and meet more of the [goals](#goals) we set up. At some point I'll extend on it, make it more idiomatic, and clean it up. It's pretty much a dirty POC at the time of writing, but nevertheless I wanted to share it.

Also note it was written for Github Enterprise, so might need a few small tweaks to work for public repos (mostly around the URLs used). Be respectful your scripts' frequency (remember, it's in the filename, e.g. `github-pr-checker.1h.sc`)

Writing scripts with Scala-cli was generally pleasant, and I will likely reach for it again when solving similar problems - however I couldn't figure out how to have a great user experience. There are [IDE setup docs for scala-cli](https://scala-cli.virtuslab.org/docs/commands/setup-ide/), which could have helped improve highlighting or autocompletion. They do acknowledge [IDE user experience could be better](https://scala-cli.virtuslab.org/docs/guides/ide/).

# But I'm on Windows / Linux

todo: have a look for alternatives that use plugins and thus could use this stuff too