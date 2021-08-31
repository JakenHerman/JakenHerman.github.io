---
layout: post
title:  "⚡ Improving your Reddit bot with Supabase"
excerpt: "In my previous post, we created a Reddit bot using Python and PRAW, but there was one issue with it. Let’s look at the code we left off with"
comments : true
date:   2021-09-01 12:00:00 -0600
categories: [Python, Bots, Reddit, API, Supabase]
---

In my (previous post)["https://www.jakenherman.com/articles/2021-08/reddit-bot-praw"], we created a Reddit bot using Python and PRAW, but there was one issue with it. Let’s look at the code we left off with:

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

What’s wrong with this code? Well, every time we run it, we will respond to the comment — even if we’ve already responded previously. This could be seriously annoying to Reddit users if our bot is just spamming them with “Where are you?” just because they said “Scooby dooby doo” in one comment.

To resolve this issue, we’re going to assume you have a Supabase account, if you don’t have one — go create one at (Supabase.io)[https://www.supabase.io] (this is free). Once you’re logged in to Supabase, create a new project by pressing the green “New project” button, and give the project a name and a strong database password. Once you press “Create new project”, Supabase will get your project ready — and this may take a minute or two to process, just be patient.

Now when you’re in your Supabase project, press on the icon with the tooltip “Table editor” on the left-hand side to set up your database that will store the IDs of comments we’ve already replied to. The button will be indicated by the icon shown below.

![Supabase Table Editor Icon](https://cdn-images-1.medium.com/max/800/1*3_KP5flNfVu0DdM5sBkzjg.png)

Press the “Create a new table” button in the center of the screen, and a modal will appear for you to create a new database table. We’ll give this a name of `comment_ids`, we’ll ignore the description for now, and ensure that RLS (Row Level Security) is enabled. By default, Supabase gives us three columns: An `id` column of type `int8`, and `created_at` and `updated_at` columns of type `timestamptz`. Reddit comment ids are strings, not integers, so we’ll change the type of the `id` column to `varchar`. We don’t care about when the comment reply was created, so we can change the `created_at` column to be named `comment_body`, and set the type to `varchar` as well, then remove the `updated_at` column altogether, it is not needed. If your form looks similar to the photo below, you’re ready to press “Save” and move on.

![The proper configuration for creating our comment_ids database table](https://cdn-images-1.medium.com/max/800/1*4XPSQyHjcx0Xz6S5_GexKA.png)

Now that are database is ready to receive comments, we need to access it in our Python code. There are two things we’ll need to access our database in our code: Our Supabase config URL and our Supabase `service_role` API secret key. To obtain these, navigate to the Supabase settings page, then press API on the left. The `service_role` API secret key will be in the Project API keys card, and the config URL will be in the Config card, as shown below:

![Access your secret API key and URL from the API setting page in Supabase](https://cdn-images-1.medium.com/max/800/1*5ARddX34jD3L_PfltLZaKg.png)

Back in your code editor, open your `.env` file, and create an environment variable for `SUPABASE_URL` that points to your Supabase config URL and a variable for `SUPABASE_KEY` that points to your Supabase service_role API key. Your final `.env` file should look something like this:

{% highlight bash %}
API_CLIENT=<YOUR_API_CLIENT>
API_SECRET=<YOUR_API_SECRET>
REDDIT_USERNAME=<YOUR_REDDIT_USERNAME>
REDDIT_PASSWORD=<YOUR_REDDIT_PASSWORD>
SUPABASE_KEY=<YOUR_SUPABASE_KEY>
SUPABASE_URL=<YOUR_SUPABASE_URL>
{% endhighlight %}

In order to access Supabase in python, we’ll need to install the `supabase-py` package by running `pipenv install supabase-py`. Once that’s installed, we can import the package into our code and create our Supabase client:

{% highlight python %}
  import os
  import praw
  from supabase_py import create_client, Client

  reddit = praw.Reddit(
    username = os.environ.get('REDDIT_USERNAME'),
    password = os.environ.get('REDDIT_PASSWORD'),
	client_id = os.environ.get('API_CLIENT'),
	client_secret = os.environ.get('API_SECRET'),
	user_agent = "Scooby Searcher Bot"
  )
  
  url: str = os.environ.get("SUPABASE_URL")
  key: str = os.environ.get("SUPABASE_KEY")
  supabase: Client = create_client(url, key)

  for comment in reddit.subreddit('cartoons').comments(limit=1000):
    if "scooby dooby doo" in comment.body.lower():
      comment.reply("Where are you?")
{% endhighlight %}

So the only lines we’ve added here to our code we wrote in part 1 are lines 3, and 13–15. Line 3 imports the package into our program, and lines 13–15 creates a Supabase client so we can interact with our database we created earlier. Now, let’s query the database to `select` all of the comments from our `comments_id` table, then create an array of the IDs we’ve already responded to:

{% highlight python %}
  import os
  import praw
  from supabase_py import create_client, Client

  reddit = praw.Reddit(
    username = os.environ.get('REDDIT_USERNAME'),
    password = os.environ.get('REDDIT_PASSWORD'),
	client_id = os.environ.get('API_CLIENT'),
	client_secret = os.environ.get('API_SECRET'),
	user_agent = "Scooby Searcher Bot"
  )
  
  url: str = os.environ.get("SUPABASE_URL")
  key: str = os.environ.get("SUPABASE_KEY")
  supabase: Client = create_client(url, key)
  
  comments = supabase.table("comment_ids").select("*").execute()
  comments_already_responded_to = [comment.id for comment in comments.get("data", [])]

  for comment in reddit.subreddit('cartoons').comments(limit=1000):
    if "scooby dooby doo" in comment.body.lower():
      comment.reply("Where are you?")
{% endhighlight %}

Lines 17 and 18 are the only bit of code added from the previous block. In line 17, we create a variable named `comments` that stores the value of the `select("*")` query on our `comment_ids` table. (`*` means to grab everything that exists within that table.)

On line 18, we’re just looping through each of the comments returned from our database query, and adding the `id` field of that `comment` to an array named `comments_already_responded_to`.

Next, we’ll modify our code to only respond to a comment if we have not already responded to it in the past. If we respond, we’re going to add the new comment id to our `comment_ids` table so the next time we run our code, we will not respond to it again:

{% highlight python %}
  import os
  import praw
  from supabase_py import create_client, Client

  reddit = praw.Reddit(
    username = os.environ.get('REDDIT_USERNAME'),
    password = os.environ.get('REDDIT_PASSWORD'),
	client_id = os.environ.get('API_CLIENT'),
	client_secret = os.environ.get('API_SECRET'),
	user_agent = "Scooby Searcher Bot"
  )
  
  url: str = os.environ.get("SUPABASE_URL")
  key: str = os.environ.get("SUPABASE_KEY")
  supabase: Client = create_client(url, key)
  
  comments = supabase.table("comment_ids").select("*").execute()
  comments_already_responded_to = [comment.id for comment in comments.get("data", [])]

  for comment in reddit.subreddit('cartoons').comments(limit=1000):
    if "scooby dooby doo" in comment.body.lower() and comment.id not in comments_already_responded_to:
      comment.reply("Where are you?")
      data = supabase.table("comment_ids").insert({"id":comment.id,"comment_body":comment.body}).execute()
{% endhighlight %}

In line 21, we added another check to our conditional. Now, not only are we looking to see if the phrase “scooby dooby doo” exists in the `comment.body`, we are also verifying that the `comment.id` does not exist in the `comments_already_responded_to` array we created on line 18.

If both conditions are met, on line 22, we reply to the comment (as we always have), but now we also insert the `id` of that comment to our `comment_ids` table with the `insert()` function. We pass in a dict that matches the structure of our database table: `{"id": comment.id, "comment_body": comment.body}`.

Now you can run your bot repeatedly without having to worry about pestering Reddit users — they’ll only get your response once. In the next post, we’ll set up and GitHub Actions workflow so you do not need to manually run your Reddit bot (after all, it’s supposed to be a bot — don’t those run on their own?)

Subscribe to me on [Medium](https://medium.com/@JakenH) for quicker access to updates.