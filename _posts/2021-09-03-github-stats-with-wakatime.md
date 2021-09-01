---
layout: post
title:  "📊 Show off your coding stats on your GitHub Profile using WakaTime"
excerpt: "A neat way to spruce up your GitHub profile is to show your followers and visitors what languages you’ve been working with, and how much coding you’ve been getting done."
comments : true
date:   2021-09-03 12:00:00 -0600
categories: [GitHub]
---

![](https://miro.medium.com/max/700/1*EAX459Bx17zD6Xxvwi9NcA.png)

A neat way to spruce up your GitHub profile is to show your followers and visitors what languages you've been working with, and how much coding you've been getting done. As you can tell from the screenshot of my GitHub profile, I've worked mostly with Python and JavaScript.

Prerequisites to following this post:

 - A GitHub account
 - A WakaTime account (free, set it up [here](https://wakatime.com))
 - A profile README (if you don't have one, you can create it by following the [GitHub docs](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-profile/customizing-your-profile/managing-your-profile-readme))

To begin, edit your `README.md` file and insert the following comments wherever you want the code stats to appear:

{% highlight yaml %}
<!--START_SECTION:waka-->
<!--END_SECTION:waka-->
{% endhighlight %}

Next, head over to https://wakatime.com and get your WakaTime API Key from your "Account Settings" in WakaTime, shown below:

![](https://miro.medium.com/max/700/1*sf0YILfutJN-FCz0MhLPmw.png)

The next thing you'll need to do is install the WakaTime plugin for whichever text editor or IDE you are using. The instructions for each of these varies, so your best bet to get it right is to follow the instructions listed on WakaTime. To see all of the supported editors and IDEs, look at [this page](https://wakatime.com/plugins). Because I'm assuming most of you will be using VS Code, these are the instructions for setting up WakaTime in VS Code: https://wakatime.com/vs-code

Now, back in your GitHub repository, go to the "Settings" tab, and click "Secrets". Create a new **Repository Secret**, not an environment secret. The name of this secret should be `WAKATIME_API_KEY`, and the value of the secret should be the API key you obtained in WakaTime earlier in the post.

Navigate to the "Actions" tab of your GitHub repository, and select "Set up a workflow yourself", as shown below:

![](https://miro.medium.com/max/700/1*_LKlZc3S-4spWjHPDtPnpg.png)

Paste the following code in the editor, and commit the changes:

{% highlight yaml %}
name: Work Stats Readme

on:
  workflow_dispatch:
  schedule:
    # Runs every 2 hours
    - cron: "0 */2 * * *"

jobs:
  update-readme:
    name: Update this repo's README
    runs-on: ubuntu-latest
    steps:
      - uses: athul/waka-readme@master
        with:
          WAKATIME_API_KEY: ${{ secrets.WAKATIME_API_KEY }}
{% endhighlight %}

As you can see, we're only running this job to check our stats every 2 hours, so adjust as you see fit (tip: https://crontab.guru is a great way to visualize crons). Keep in mind, it may take time for your stats to appear (you need to log enough time in WakaTime, and give it enough data to actually report).

Hope you've enjoyed, comment below with your GitHub repositories if you tried this out, I'm looking forward to seeing all the stats 😊


Subscribe to me on [Medium](https://medium.com/@JakenH) for quicker access to updates.