---
title: "Send tasks & ideas to your work computer from anywhere, any time"
date: 2019-01-01
lastmod: 2019-03-17
draft: false
garden_tags: ["productivity"]
summary: "… Increasing your productivity, autonomy, and enjoyment at work"
status: "evergreen"
---

Do you ever have an amazing idea for work whilst away from your work PC, but hate mingling work tasks/ideas into your personal to-do? Do you forget some of these great ideas, or can't conveniently access them at work?

This guide shows how you can fix these problems with the free app Todoist. The following setup has worked incredibly well for me, and I hope you can benefit from it too!

# Our goals
1. Our two widgets will look like this:

{{< figure src="widget_view.jpg" width="50%" >}}


2. We can send tasks from a phone to a work project like this: 

{{< figure src="add_task.jpg" width="50%" >}}

3. I want this *work widget* to include only:
    - **work project** tasks, and 
    - tasks for **today or overdue**. I don't want to see tasks I scheduled for next year!

4. I want my personal widget to EXCLUDE these work tasks.

# The setup

First create a "Work" project on your personal account. Invite your work email address to this project. You can use the [Todoist app](https://medium.com/r/?url=https%3A%2F%2Ftodoist.com%2Fdownloads) for Desktop, Mobile, Browser, or Wearables.

For the filter the [Todoist guide](https://medium.com/r/?url=https%3A%2F%2Ftodoist.com%2Fhelp%2Farticles%2Fintroduction-to-filters) actually lists the filter query we'll need as their first example:

> (today | overdue) & #Work


Todoist comes with many *filters* already, but a limit of three means that we need to either delete existing filters, or subscribe. I deleted the existing filters - I don't need them. So, create a "work filter":

{{< figure src="add_filter.jpg" width="50%" >}}

Then, create a "personal filter" too. As you may have guessed we'll be using this query: 

> (today | overdue) & !#Work

Now, create your Todoist *widget* and place it somewhere in your launcher. I recommend making a launcher page specific for work, which only has apps you need to use for work - e.g. 2FA apps, and this widget. This will help separate work and private life.

Enter the widget's settings, and under "Choose view" select our new "Work filter":

{{< figure src="widget_settings.jpg" width="50%" >}}

For our personal Todoist widget, we need create another widget but this time we choose our "Personal filter". I recommend having this on a launcher page with other organisational tools, such as a widget of your Calendar in Week/Month view.

Wherever you are, you can now hit the ➕ icon on a Todoist widget to create a **#work** task - which you can interact with on your work machine!

# Closing notes

One last piece of advice to make this flow even better: Todoist is pretty smart, so to make a weekly reminder every 4pm on Friday you write a task like shown at the start of the article. You can read more about Todoist's quick add ability [here](https://medium.com/r/?url=https%3A%2F%2Ftodoist.com%2Fhelp%2Farticles%2Ftask-quick-add).

Here's to working with lots of interesting ideas at work in 2022 - wherever and whenever you get those ideas!