￼# Recent Tweets 1 Challenge
We're going to build a simple application that fetches the most recent tweets from a given Twitter username.
There should be two types of URLs: the index page with a URL field to enter a Twitter username and a URL to display the most recent tweets of a particular username.
That is,
```
http://localhost:9393/jfarmer
```
should display the most recent tweets from https://twitter.com/jfarmer.
The goal of this challenge is to become familiar with working with third-party APIs and the kind of architecture decisions necessary to support that. We'll add support for more API endpoints later.
##Objectives
###Your First Twitter Application
Add the [Twitter gem](http://rdoc.info/gems/twitter) to your Gemfile and run
```
bundle install
```
to install the Gem. Read the Configuration section on the Twitter gem's GitHub page.
You'll have to register a Twitter application on Twitter and get an API key and API secret. You can do this at [https://dev.twitter.com/apps/new](https://dev.twitter.com/apps/new) .
This will also be your first [OAuth-based application](https://dev.twitter.com/docs/auth/oauth/faq) . OAuth is a standardized authentication protocol that allows a web application to delegate authentication to a third- party, e.g., "Log in via Twitter," "Log in via Facebook," "Log in via Google," etc.
We don't be supporting "Log in via Twitter" yet, so when you go to create a Twitter application the only fields that matter are the application name (which must be unique across all Twitter applications) and application description. The application URL can be anything and the callback URL can be blank.
Note: You'll need a callback URL in a world where you want to support "Log in via Twitter."
After creating your application you'll be redirected to your application configuration page. The URL should look like
￼￼￼￼￼￼￼￼￼
￼```
https://dev.twitter.com/apps/<#application ID>/show
```
At the bottom of the page you'll see a section called Your access token, which looks roughly like this:
[http://cl.ly/image/340S2F2t0V3Q](http://cl.ly/image/340S2F2t0V3Q) . Create an access token for yourself. You now have all the information you need to build a Twitter client. Follow the directions in the
configuration section of the Twitter gem.
Here's a simple test of whether you understand how the Twitter gem and API work and whether your environment is set up correctly. Can you write a simple command-line Ruby program — no more than 5-10 lines — to tweet something from the command line on your Twitter account?
If you want to use rake console you'll have to require 'twitter' and configure the Twitter gem in your environment.rb file. While you can require these keys directly in your environment file, this is not a good idea if you are uploading your applications to GitHub or otherwise making this code (and your keys) public. To avoid this, you can put your keys in a yaml file and load it in your environment.rb file and then put this yaml file in your .gitignore file so you can access it locally but it will not be uploaded to GitHub. See [this post](https://gist.github.com/dbc-challenges/c513a933644ed9ba2bc8)
###Recent Tweets (not cached)
Create a routes that looks like this: ```
get '/' do end
get '/:username' do end
```
Make /:username display the 10 most recent tweets of the supplied Twitter username. Edit
environment.rb to add the appropriate configuration.
Don't worry about leaking your development credentials into the public for now.
###Recent Tweets (cached)
The above URL will be pretty slow. Every time you access it you have to make an API request, which could take a second or more. Let's create a local cache of the results so it's only slow the first time we get a list of recent tweets.
Create models Tweet and TwitterUser. Implement something like the following: ```
get '/:username' do
@user = TwitterUser.find_by_username(params[:username])
￼￼￼￼￼￼
￼if @user.tweets.empty?
# User#fetch_tweets! should make an API call
# and populate the tweets table #
# Future requests should read from the tweets table
# instead of making an API call @user.fetch_tweets!
end
@tweets = @user.tweets.limit(10) end
```
Y our code doesn't have to literally look like the code above, although the above is a solid foundation. Y ou will not be penalized if you write something different. This will not count towards your final grade.
###Recent Tweets (cached + invalidation)
The nice thing about the cached version is that only the first request is slow. The bad thing about the cached version is that the list of tweets quickly becomes stale. If there's any data in the database we use that data, even if it's two years old.
We need to flag when the cache is stale and re-fetch the data if it's stale. Let's say for now that the cache is stale if we've fetched the recent tweets within the last 15 minutes. Change your controller code to work thus:
```
get '/:username' do
@user = TwitterUser.find_by_username(params[:username]) if @user.tweets_stale?
# User#fetch_tweets! should make an API call
# and populate the tweets table #
# Future requests should read from the tweets table
# instead of making an API call
@user.fetch_tweets! end
@tweets = @user.tweets end
```
The logic about what a "stale tweet" means should be in the TwitterUser model.
###Fancier Invalidation
Can you think of a better way to invalidate the cached tweets? That is, decide when the data is stale and needs to be re-fetched?
A famous saying goes: ["There are only two hard things in computer science: cache invalidation and naming things."](http://martinfowler.com/bliki/TwoHardThings.html)
￼￼￼￼One issue is that every user shouldn't have their cache refreshed on the same schedule. Someone who
￼tweets once a year doesn't need to have their cache refreshed every 15 minutes.
Can you modify the User#tweets_stale? method to do something fancier? Maybe look at the average time between the last N tweets and use that as the "stale" threshold on a per-user basis?
Think of a few possibilities and discuss the pros and cons with your pair. Implement one.
