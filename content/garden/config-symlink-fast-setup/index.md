---
title: "Symlinks, syncs, and app configuration"
date: 2023-01-18
lastmod: 2023-01-18
draft: false
garden_tags: ["tech", "applications"]
summary: "Save time when setting up your apps (again)"
status: "growing"
---

When setting up a new Windows device, there are some tools that can help you to install your apps quickly and easily from the command line (WinGet, chocolatey). Whilst these are great (and can actually install most of your apps), one thing they can’t do is transfer your configuration between devices.

[//]: #  (TODO link to a blog on what I found about export/import apps WinGet, chocolatey, etc.)
[//]: # ([Essential Windows setup]&#40;https://www.notion.so/Essential-Windows-setup-06c52a2fb84141dd8ab7bf89f7bb2083&#41;)

In this short blog, we’ll be using:

- A cloud sync of choice (Google Drive / OneDrive / DropBox etc.)
    - So I have my configuration files on any device
- Symbolic links, in a tiny script file
    - So apps can easily access the backed-up configuration, and sync changes
- An application we love, which uses simple configuration files

--- 

For this worked example, we'll use Espanso, and a folder "Junction" symlink.

The folder linking must be done before the source folder (Espanso's config) is created. Either do this before installing the app, or do some file/folder shuffling.
It doesn't matter if the target exists or not.

In your cloud-sync'd folder (`%HomePath%\OneDrive\Apps\espanso`), create a file `symlink.bat` and add:

```
mklink /J %APPDATA%\espanso %HomePath%\OneDrive\Apps\espanso
```

In Windows, the `%APPDATA%` folder will look like this if the command succeed; note the little arrow on our symlinked folder:

![Untitled](symlink_folder_icon.png)

In `......./OneDrive/apps/espanso/match/base.yml`, add this to the main `matches` body:

```
- trigger: ":synctest"
  replace: "your espanso config was sync'd sucessfully!"
```

Install espanso with default settings

Espanso startup logs show our symlink folder being loaded in. It does this anyway (nothing special is happening to the file loading... other than it actually loading from our cloud sync'd folder!):

```scala
... reading configs from: "C:\\Users\\james\\AppData\\Roaming\\espanso"
... reading packages from: "C:\\Users\\james\\AppData\\Roaming\\espanso\\match\\packages"
```

So if we enter our special phrase, `:synctest`, we should see it expanded to:

`your espanso config was sync’d sucessfully!`


On another computer (or a fresh installation of Windows), all you have to do is double-click the .bat file.
In just a few minutes, we've made a system that will always take care of our favourite Espanso packages & personalised keywords.
Also note that any changes we make will be backed up to the cloud too!

---

What we’ve done here can be applied to more applications which have simple configuration. 
The scripts to create junctions/symbolic links could be per-app, or you could put them all in one script (to be run before a big WinGet/Brew install).

Here’s a few configurations ones I may set up this synchronisation for:

- Headphone EQ - Peace/EqualizerAPO - .txt files represent frequencies/decibels etc.
- GPU Overclock/Undervolt - MSI Afterburner - Custom curve is a bit tricky to make (I have to search for my notes on instructions every time I set up)
- ... I can't think of more, but I'm happy even with just Espanso having this :-)

The same principles apply to Ubuntu and MacOS - but implementation of your symlink script files might look more like [this StackExchange answer](https://apple.stackexchange.com/a/115647).
