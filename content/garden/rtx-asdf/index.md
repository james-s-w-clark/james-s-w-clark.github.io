---
title: "RTX ON - it's execution time"
draft: false
garden_tags: ["Tooling]
summary: "... it's never been easier to get set up"
status: "evergreen"
date: 2023-07-21
lastmod: 2023-07-21
---

I've been using previously used `sdkman` for a few years to manage my JDK & Scala installations. It supports a good amount of tooling, but it's very JVM focused. You may know `nvm` for managing and switch Node versions - it's similar.

Recently I've been getting into Elixir & Phoenix LiveView, and I came across a similar tool called `asdf`. Actually though, thanks to its "plugins" system and almost 700 plugins, you can install so many different tools.

This was working great for Elixir & Erlang, but the ergonomics felt a little off. In order to list versions, you have to first download the plugin. And due to its "shim" mechanism, it adds about 100ms delay to each command that passes through the asdf executable (my ELI5 understanding).

I then came across [rtx](https://github.com/jdxcode/rtx), a Rust tool inspired by `asdf` that takes a different approach. Here's some features I'm really liking:
- `rtx` points to tooling versions via changes to the PATH - this keeps interactions fast
- Local/Global installs - If you have a bunch of microservices on different Node/Java versions, `rtx` reloads the PATH when you open a project in your terminal. You don't need to run commands like `nvm use node 16` - it's automatic
- `asdf`'s amazing plugins - but you don't have to explicltly install them first!
- documentation - the CLI & interactions are friendly, and setup is (almost) frictionless
- .rtx.toml configuration - easy to use, and really powerful - see below!


For our documentation website, I suggested we move from install nvm/node/yarn/sbt to just this configuration file (.rtx.toml):

```
[tools]
node = "16"
yarn = "1.22.19"
sbt = "1.9.2"
```

and this one-click script (in IntelliJ) to go from 0 to READY:

```shell
curl https://rtx.pub/install.sh | sh # install rtx 
echo 'eval "$(~/bin/rtx activate zsh)"' >> ~/.zshrc # hook rtx into shell
rtx install # install tooling
yarn # install deps 
yarn run # launch dev website
```

That's a nice developer experience. I'm happy to know someone can clone a repo, click a button, grab a drink, and come back to a website!

---

# CI - does it add value here?

So, `rtx` is pretty cool for local development - but what about CI?

For our main project, we use a JDK, Scala, and Mill. 
There's a few Actions for setup (setup-java, coursier-setup, mill-setup, etc.) - but they usually want a version typing out. This could lead to drift between development and CI, and introduce a bit of toil when somebody finally notices or remembers.

With the `rtx` Action, our .rtx.toml files can be used to set everything up in less code (probably just one line, the "uses", as it should spot the .rtx.toml file automatically!), and it'll stay up to date.

# ... but my versions for different tools are scattered around my source!

Let's say you use three tools, which are specified in different files:

- openjdk-17 - in a Dockerfile
- Scala 2.13.xy - in a Dependencies.sc file
- mill - in .mill-version

Fortunately, `rtx` uses the `tera` templating engine so we can grab these dynamically.
These commands are kinda grim (I couldn't use " or '; I found `cut` to be a good command, thanks ChatGPT), but are probably "good enough" to not need updating. The sources they read won't be changing spacing much:

```toml
[tools]
mill = { version = "{{exec(command='echo $(cat .mill-version)')}}" }
java = { version = "{{exec(command='grep -m 1 openjdk docker/Dockerfile | cut -c 12- | tr : -')}}" }
scala = { version = "{{exec(command='grep -m 1 2.13 dependencies/Dependencies.sc | cut -c 33-39')}}" }
```

The secret sauce here is:
- `grep -m 1 <phrase>` returns the first line that matches
- `cut 12-` gives from the 12th char onwards, `cut 33-39` does what you'd think

Yep, it does look dumb, but:
- For Java we just pin to a major version; if we stick to the same vendor, there'll be no issue
- For Mill, it's just a plain cat. Not too bad :)
- For Scala, until there's a migration to Scala 3 then we'll just see 2.13.11 -> 2.13.xy

# Conclusion

I encourage you to give [rtx](https://github.com/jdxcode/rtx) a try. It'll be my tooling manager of choice for personal projects now, and I'm encourgaging its use at work.