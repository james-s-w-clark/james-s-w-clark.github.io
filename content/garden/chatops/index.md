---
title: "ChatOps - just say the word"
draft: false
garden_tags: ["Ways-Of-Working"]
summary: "... WoW, I can use some of these posthaste!"
status: "seeding"
date: 2023-05-18
lastmod: 2023-05-18
---

ChatOps: you say some trigger, and you get some response. The processing behind the scenes can be as complex or niche as you like.

In this blog, we'll talk about three different ChatOps tools (Slack, GitHub Actions, Hubot) and how they can:
- Set up a basic reminder (Slack)
- Trigger image builds, performance tests, etc. on PRs (GitHub Actions)
- Send a list of open PRs, and their review counts, to Slack (Hubot)

# Slack

Slack has a few [baked in commands ("Shortcuts")](https://slack.com/intl/en-gb/help/articles/360057554553-Use-shortcuts-to-take-actions-in-Slack).
The most useful I've seen is reminders - whether for yourself or for your team. Here's a few examples, with the format []`/remind [yourself or #channel] [what] [when].`](https://slack.com/intl/en-gb/help/articles/208423427-Set-a-reminder):

- /remind #my-team to join [Google Meet](https://meet.google.com/?pli=1) on Wednesday at 4:30pm
- /remind me to file TPS reports in 20 minutes
- /remind me to have a great weekend every Friday at 5pm





# GitHub Actions

Note: If you have a project on Public GitHub, you can use their action "runners" for free. If you're in the GitHub Enterprise Suite, you'll need to deploy your own [`runs-on: [self-hosted]`](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-self-hosted-runners-in-a-workflow#using-default-labels-to-route-jobs) "runners". There are some projects that can help you kick-start self-hosting, such as [the scalable spot instance setup
"terraform-aws-github-runner".](https://github.com/philips-labs/terraform-aws-github-runner)

perf tests awkward process, evolving over time as we change our Jenkins pipelines to multibranch etc.

very valuable - perf test before merge

low barrier to entry: chat ops. Ask a PR to be perf tested, and an hour later you have 

1. docker images build
2. applications performance tested

# Hubot