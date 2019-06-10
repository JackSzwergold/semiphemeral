![Logo](/img/logo-small.png)

# Semiphemeral

There are plenty of tools that let you make your Twitter feed ephemeral, automatically deleting tweets older than some threshold, like one month.

Semiphemeral does this, but also lets you automatically exclude tweets based on criteria: how many RTs or likes they have, and if they're part of a thread where one of your tweets has that many RTs or likes. It also lets you manually select tweets you'd like to exclude from deleting.

~~It can also automatically delete your old direct messages.~~ (DM support is currently [broken](https://github.com/tweepy/tweepy/issues/1081) in tweepy, I'm gonna wait until it's fixed first.)

_Read more in the blog post: [Semiphemeral: Automatically delete your old tweets, except for the ones you want to keep](https://micahflee.com/2019/06/semiphemeral-automatically-delete-your-old-tweets-except-for-the-ones-you-want-to-keep/)_


## Installation

```
pip3 install semiphemeral
```

## How it works

Semiphemeral is a command line tool that you run locally on your computer, or on a server.

```
$ semiphemeral
Usage: semiphemeral [OPTIONS] COMMAND [ARGS]...

  Automatically delete your old tweets, except for the ones you want to keep

Options:
  --help  Show this message and exit.

Commands:
  configure  Start the web server to configure semiphemeral
  delete     Delete tweets that aren't automatically or manually excluded
  fetch      Download all tweets
  stats      Show stats about tweets in the database
```

Start by running `semiphemeral configure`, which starts a local web server at http://127.0.0.1:8080/. Load that website in a browser.

You must supply Twitter API credentials here, which you can get by following [this guide](https://python-twitter.readthedocs.io/en/latest/getting_started.html). Basically, you need to login to https://developer.twitter.com/ and create a new "Twitter app" that only you will be using (when creating an app, you're welcome to use https://github.com/micahflee/semiphemeral as the website URL for your app).

From the settings page you also tell semiphemeral which tweets to exclude from deletion:

![Settings](/img/settings.png)

Once you have configured semiphemeral, fetch all of the tweets from your account by running `semiphemeral fetch`. (It may take a long time if you have a lot of tweets -- when semiphemeral hits a Twitter rate limit, it just waits the shortest amount of time allowed until it can continue fetching.)

Then go back to the configuration web app and look at the tweets page. From here, you can look at all of the tweets that are going to get deleted the next time you run `semiphemeral delete`, and choose to manually exclude some of them from deletion. This interface paginates all of the tweets that are staged for deletion, and allows you to filter them by searching for phrases in the text of your tweets.

Once you have chosen all tweets you want to exclude, you may want to [download your Twitter archive](https://help.twitter.com/en/managing-your-account/how-to-download-your-twitter-archive) for your records.

Then run `semiphemeral delete` (this also fetches latest tweets before deleting). The first time it might take a long time. Like with fetching, it will wait when it hits a Twitter rate limit. Let it run once first before automating it.

After you have manually deleted once, you can automatically delete your old tweets by running `semiphemeral delete` once a day in a cron job.

Settings are stored in `~/.semiphemeral/settings.json`. All tweets (including exceptions, and deleted tweets) are stored in a sqlite database `~/.semiphemeral/tweets.db`.

## Deleting old likes

The Twitter API is only willing to tell you about your last 4000 likes. If you've already tried to fetch and delete your likes, but still have a lot of old likes, you can use semiphemeral to automate unliking them.

_**WARNING: One does not simply unlike old tweets.** According to the Twitter API, you didn't like these old tweets, so you can't unlike them -- even though they're listed in your like history. The only way to remove them from your like history is to LIKE THEM AGAIN, and then you can unlike them. This is very noisy. Every time you re-like a tweet, the user will get a notification._

In order to get a list of all of your old likes (since the Twitter API won't give it to you), you must go to https://twitter.com/settings/your_twitter_data and download your Twitter data (note that this is different than your "Twitter archive", which doesn't include information about your likes). Twitter will email you a link to a zip file. When you unzip it there will be many files, including a file called `like.js`. Run this command, with the path to your `like.js`, for example:

```sh
semiphemeral unlike --filename ~/Downloads/twitter-2019-06-07-8195574bc935602c0056aee12fb11de78553835ace755eb782c895283f7fa14e/like.js
```

Your filename will be different than this one, so make sure you update the command to match it.

_**WARNING: Don't automate this command.** Pay close attention when you run this command, and after it completes successfully don't run it again. Otherwise you'll relike and unlike all of these tweets over again. If it fails in the middle of the like/unlike process, then download a new copy of your Twitter data and run it again using the new `like.js` instead._

First this will fetch all of the old tweets you liked a long time ago, relike them, and then unlike them again. Every relike will cause a notification, but at the end of the process your likes will have actually been deleted.

New likes don't have this problem, so as long as you regularly run `semiphemeral delete`, your new likes will automatically get deleted.

## Development

Make sure you have [pipenv](https://pipenv.readthedocs.io/en/latest/). Then install dependencies:

```sh
pipenv install --dev
```

And run the program like this:

```sh
pipenv run python ./app.py --help
```
