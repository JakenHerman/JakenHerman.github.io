---
layout: post
title:  "🔧 Using GitHub Actions to run your Reddit Bot"
excerpt: "In this post, we’ll go over how to run your Reddit bot at a certain time interval using GitHub Actions."
comments : true
date:   2021-09-02 12:00:00 -0600
categories: [Python, Bots, Reddit, API]
---

In this post, we’ll go over how to run your Reddit bot at a certain time interval using GitHub Actions. While you could host your bot on Heroku or some other hosting service, there may not be a need for that if your bot is just a simple reply-bot like ours is. If you aren’t caught up in this series, I’d first recommend you check out my post on [creating a Reddit bot with Python and PRAW](https://medium.com/@JakenH/creating-a-reddit-bot-with-python-and-praw-185387248a22), then moving on to the post where I show you how to [improve your reddit bot with Supabase](https://medium.com/@JakenH/improving-your-reddit-bot-with-supabase-68b0fd847273). If you follow those two posts, you’ll be caught up and ready to go on this one.

What you’ll need:

 - A Reddit Bot, preferably the one we created in the first two posts of this series
 - A GitHub Account

Let’s start by creating a GitHub repository for our bot, and getting our code pushed to this repository. To start, go to https://www.github.com and press “New” on the left-hand side:
![Creating a new repository on GitHub](https://miro.medium.com/max/700/1*MNV8QCoQI1S5SuHLYHGl7A.png)

Next, in the input box for “repository name”, type the name of your Reddit Bot, make the repository either public or private, then press the green “Create repository” button to get started. Now our repository is ready to receive our code.
There’s one very important step we need to make sure we have in place first, and that is to tell Git to ignore our `.env` file, otherwise we’d be posting our Reddit API secret keys and our Supabase secret keys to the public! Open terminal on your machine and, assuming your bot is named “scooby-searcher” in your “Documents” directory, run `cd ~/Documents/scooby-searcher`. Next we’ll run `vim .gitignore`. This will create a `.gitignore` file and open it in the [Vim](https://www.vim.org) text editor. Press the “i” key on your keyboard, which will allow you to start inserting text into the file. All we need to add is “.env”. After typing, press Esc on your keyboard followed by `:wq`. This tells Vim to “save and quit”. Now Git will ignore our `.env` file.

Next, let’s initialize a Git repository locally within our scooby-searcher directory by typing `git init`. You should get a message in your terminal something like `Initialized empty Git repository in Documents/scooby-searcher.git`. Now we need to point our local repository to our remote repository we just created on GitHub. To do this, in your browser navigate to the GitHub repository you just created and copy the URL. Now go back to your terminal and run `git remote add origin https://github.com/path_you_just_copied.git`. Our local instance is now hooked up to our remote repository on GitHub. To add our files to GitHub, let’s run the following chain of commands:

 * `git add .` , which tells Git to add all of the files that do not match a pattern in our .gitignore file.
 * `git commit -m 'Initial commit'`, which tells Git to commit the staged changes with a message of “Initial commit”
 * `git push origin master`, which will push our changes to GitHub.

Now go back to the repository on your GitHub account, refresh the page, and 🎉 Voila 🎉, your code is there. But just having your code in a GitHub repository will not make your bot run, there’s a few more steps we need to take. From your GitHub repository, click on the “Settings” tab for that repository, then in the left-hand menu select the “Secrets” option. Once you’re there, you’ll select “New repository secret”. You’ll create a new repository secret for each of the environment variables in your `.env` file (`SUPABASE_URL`, `REDDIT_PASSWORD`, etc.). Match the casing exactly as you have in your `.env` file. One important thing here is you must create a **repository secret**, not an environment secret. Your input should look something like this while creating:

![](https://miro.medium.com/max/700/1*X0It3_wqlO5ibIi8jtFgMg.png)

And like this when you’re done:

![](https://miro.medium.com/max/700/1*-qgu0pH8toOcm3O-XcJo0g.png)

Great, now we’re ready for the final step — creating the Action to run our bot. From your GitHub repository, select the “Actions” tab. There will be a few suggested premade workflows suggested to you, but we’ll create our own. To do this, select the “set up a workflow for yourself” text, as shown below:

![](https://miro.medium.com/max/700/1*_LKlZc3S-4spWjHPDtPnpg.png)

Now, I’m just going to dump our entire .yml file here, and I’ll explain the important bits of it to you afterwards:

{% highlight yaml %}
name: Run bot

on:
  workflow_dispatch:
  schedule:
    # Runs every 2 hours
    - cron: "0 */2 * * *"

jobs:
  prep:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.9
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    - name: Install pipenv
      run: |
        python -m pip install --upgrade pip
        pip install pipenv
    - name: Install dependencies
      run: |
        pipenv install
    - name: Create ENV file
      shell: bash
      env:
        SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
        SUPABASE_KEY: ${{ secrets.SUPABASE_KEY }}
        REDDIT_USERNAME: ${{ secrets.REDDIT_USERNAME }}
        REDDIT_PASSWORD: ${{ secrets.REDDIT_PASSWORD }}
        API_SECRET: ${{ secrets.API_SECRET }}
        API_CLIENT: ${{ secrets.API_CLIENT }}
      run: |
        touch .env
        echo REDDIT_USERNAME=$REDDIT_USERNAME >> .env
        echo REDDIT_PASSWORD=$REDDIT_PASSWORD >> .env
        echo API_SECRET=$API_SECRET >> .env
        echo API_CLIENT=$API_CLIENT >> .env
        echo SUPABASE_KEY=$SUPABASE_KEY >> .env
        echo SUPABASE_URL=$SUPABASE_URL >> .env
        cat .env
    - name: Run bot
      shell: bash
      run: |
        pipenv run python bot.py
{% endhighlight %}

Line 1 is just what we’re naming this workflow — in this case “Run bot”. You can name this workflow whatever you’d like.
On lines 3 through 7, we’re specifying how this action will run, and we’re providing two different methods. One is `workflow_dispatch`, which means we can literally go to the GitHub Actions tab of our repository and say “Run this workflow now”. The other method is `schedule`, which means “run this workflow on a specified schedule”, in our case the bot will run every 2 hours (`0 */2 * * *`). If you want to change the scheduled interval that your bot runs on but you’re not comfortable with crons, there’s a great tool to visualize your intervals at https://crontab.guru.

Lines 9 through 46 are our actual jobs that will be running. Everywhere you see a line start with `— name` is a different job. The first of these, “Set up Python 3.9” is a pretty simple one, not much explanation needed there. The second job installs pipenv, similar to the way we installed it on our machine locally. You can see the two commands this job runs are `python -m pip install --upgrade pip`, followed by `pip install pipenv`. The third job installs the dependences listed in our `Pipfile` of our project by running the command `pipenv install`. This installs our `supabase-py` package as well as `PRAW`. The fourth job creates a temporary `.env` file similar to the one we created on our local machine but did not push. Don’t worry, GitHub hides your keys still, so these will not be leaked. This is why we set up the GitHub secrets earlier in the post, and you can see that we are accessing them via `${{ secrets.KEY_NAME }}` in our job. The fifth and final job runs our command that we used to run our bot locally: `pipenv run python bot.py`.

Once you’ve input this code into the GitHub secrets editor, commit it to your repository by pressing the green “Start commit” button in the upper-right corner followed by “Commit new file”. Your bot will now run every two hours using GitHub Actions, totally free.
If you want manually trigger a run of your bot, you can navigate to your repository’s “Actions” tab, and under the “Workflows” section on the left-hand side, press “Run bot” to open the workflow run history. To set off the `workflow_dispatch` event trigger, press the “Run workflow” dropdown, followed by “Run workflow” again:

![](https://miro.medium.com/max/700/1*l5LGsGHENaoyYEnz0zy77A.png)

I genuinely hope you’ve enjoyed creating your Reddit bot and that you’ve found this series helpful. Thank you for reading.
Subscribe to me on [Medium](https://medium.com/@JakenH) for quicker access to updates.