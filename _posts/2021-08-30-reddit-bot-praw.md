---
layout: post
title:  "🤖 Creating a Reddit Bot with Python and PRAW"
excerpt: "In this post, we will be creating a Reddit bot using the Python programming language and the PRAW package, which stands for “Python Reddit API Wrapper”."
comments : true
date:   2021-08-30 12:00:00 -0600
categories: [Python, Bots, Reddit, API]
---

In this post, we will be creating a Reddit bot using the Python programming language and the PRAW package, which stands for “Python Reddit API Wrapper”. Our bot will scan the subreddit “/r/cartoons” for comments containing the phrase “Scooby Dooby Doo” (case insensitive), and reply to any comments containing that phrase with “Where are you?”, which will surely plant an [earworm](https://www.youtube.com/watch?v=eZXg6Uaxd2k) in the reader of the comment. There are a few prerequisites before we begin. You will need:

 - A Reddit Account
 - Python installed on your machine
 - Basic understanding of how Reddit works
 - Basic understanding of Python
 - A Supabase Account (This is free)

To begin, the first thing we need to do is create a Reddit app. Navigate to https://www.reddit.com/prefs/apps/, and scroll to the bottom of the page until you see the button that says “are you a developer? create an app…”, and click it to create a new Reddit app:

![Reddit’s “Create an app” button for developers](https://miro.medium.com/max/408/1*cILXiOSgB9UYbIMBPGgrEA.png)

Once the button has been pressed, you’ll see a form to fill out that will ask for the name of your app, what type of app you’ll be creating, a description, and some URLs. I would highly suggest you come up with a different concept than Scooby Doo for your app, but you can follow mine as a template. We will be creating a `script` app. For the "about url" and "redirect uri" you can just put in your personal website link.

![The “Create Application” form should have your app name, description, type, and urls](https://miro.medium.com/max/700/1*wbLla29Uu5miYV9v_ioUyw.png)

Once you press “create app”, your app will be listed with a client key and a secret key, shown below. Note, I’ve redacted mine — yours will have a string of characters and numbers. Do not share these with anyone. Make note of these, or at least remember how to access them again later in this post.

![Your app in Reddit should now have a personal use script, a secret key, and your app’s other information.](https://miro.medium.com/max/700/1*9DbNh0xXUkKqECGQe3j8iw.png)

Next, create a Python project on your local machine. I’m on a Mac, and I will be creating my project in my `~/Documents` directory, like so:

`cd ~/Documents && mkdir scooby-searcher && cd scooby-searcher`

Let’s use pipenv to manage packages and our virtual environment. To set pipenv up running Python 3.8, run the following command (If you don’t have pipenv yet, just run `pip install --user pipenv` (or `brew install pipenv` if you’re on a Mac):

`pipenv --python 3.8`

This is where we’ll need to get the client and secret keys back out. We’re going to create a `.env` file that will hold this information so that we can access these keys in our Python code. We are using using `.env` file variables instead of just hard-coding the strings directly in our Python code, because we’re going to place this file in our `.gitignore` so it is not tracked. This will stop us from inadvertently checking in our secret keys to our repository. You may be wondering — how will our hosted code know what the keys are in order to run? Not to worry, we’ll set up GitHub secrets and actions in a later post. Open your scooby-searcher directory in a text editor of your choice, then create a new file in the root of your project named `.env` with the following contents:

{% highlight bash %}
API_CLIENT=<YOUR_API_CLIENT>
API_SECRET=<YOUR_API_SECRET>
REDDIT_USERNAME=<YOUR_REDDIT_USERNAME>
REDDIT_PASSWORD=<YOUR_REDDIT_PASSWORD>
{% endhighlight %}

Now that we have our keys and our credentials in our project, we need some way to actually use them. To do this, we’ll need to install the PRAW package by running `pipenv install praw`.

With PRAW installed, we can create a new python file called `bot.py` (any filename will do, really) and start our file off by “Logging in” to Reddit — via code:

{% highlight python %}
  import os
  import praw
  
  reddit = praw.Reddit(
    username = os.environ.get('REDDIT_USERNAME'),
	password = os.environ.get('REDDIT_PASSWORD'),
	client_id = os.environ.get('API_CLIENT'),
	client_secret = os.environ.get('API_SECRET'),
	user_agent = "Scooby Searcher Bot"
  )
{% endhighlight %}

As you can see, logging in is extremely simple. We just call `praw.Reddit()` with the parameters `username`, `password`, `client_id`, `client_secret`, and `user_agent`, each of which uses the `os.environ.get()` function to access the variables stored in our `.env` file. The `user_agent` parameter is simply the name of our bot.

Next, let’s read some comments:

{% highlight python %}
  import os
  import praw
  
  reddit = praw.Reddit(
    username = os.environ.get('REDDIT_USERNAME'),
    password = os.environ.get('REDDIT_PASSWORD'),
	client_id = os.environ.get('API_CLIENT'),
	client_secret = os.environ.get('API_SECRET'),
	user_agent = "Scooby Searcher Bot"
  )

  for comment in reddit.subreddit('cartoons').comments(limit=1000):
    print(comment.body)
{% endhighlight %}

As you can see, it’s as simple as accessing the subreddit “cartoons” by using the `subreddit()` function on our reddit instance, then chaining a function call to `comments()` off of the return of the `subreddit()` function call. We’re limiting the result to 1,000 comments, but feel free to change this number up. Shall we respond if we find a comment that matches “Scooby Dooby Doo”? Yes, let’s:

{% highlight python %}
  import os
  import praw
  
  reddit = praw.Reddit(
    username = os.environ.get('REDDIT_USERNAME'),
    password = os.environ.get('REDDIT_PASSWORD'),
	client_id = os.environ.get('API_CLIENT'),
	client_secret = os.environ.get('API_SECRET'),
	user_agent = "Scooby Searcher Bot"
  )

  for comment in reddit.subreddit('cartoons').comments(limit=1000):
    if "scooby dooby doo" in comment.body.lower():
        comment.reply("Where are you?")
{% endhighlight %}


All we’ve added here is a check to see if `comment.body` (modified to be all lower-case) contains the phrase “scooby dooby doo”, and if so, use the `reply()` function on the comment that we access in our for-loop. Pretty neat, right?

To run your code, run the command `pipenv run python bot.py`, and enjoy the results. In the next post, I’ll walk you through how to be sure you’re not responding to a comment that you’ve already responded to, so that subsequent runs don’t pester poor Reddit users too much. To do this, we’ll create a Supabase database to store comment ids that we’ve already responded to, and add a check to the code to see if that id exists in the database before replying to comments. We’ll also go over how to use GitHub secrets and actions so your bot can run on a timer instead of you having to manually run your code.

Subscribe to me on [Medium](https://medium.com/@JakenH) for quicker access to updates.