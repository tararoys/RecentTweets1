#Recent Tweets 1 Challenge

We're going to build a simple application that _fetches_ the most recent _tweets_ from a given _Twitter username_.

There should be _two types of URLs_: the _index page_ with a _URL field_ to _enter a Twitter username_ and a _URL to display_ the _most recent tweets_ of a particular username.

That is,

```
http://localhost:9393/jfarmer
```

should display the most recent tweets from https://twitter.com/jfarmer.

The goal of this challenge is to become familiar with working with _third-party APIs_ and the kind of _architecture_ decisions necessary to support that. We'll _add support_ for more _API endpoints_ later.

##Objectives

###Your First Twitter Application

Add the [Twitter gem](http://rdoc.info/gems/twitter) to your _Gemfile_ and run

```
bundle install
```

to _install the Gem_. Read the _Configuration_ section on the _Twitter gem's GitHub_ page.

You'll have to _register_ a _Twitter application_ on _Twitter_ and get an _API key_ and _API secret_. You can do this at [https://dev.twitter.com/apps/new](https://dev.twitter.com/apps/new) .

This will also be your first [OAuth-based application](https://dev.twitter.com/docs/auth/oauth/faq) . _OAuth_ is a _standardized authentication protocol_ that allows a _web application_ to _delegate authentication_ to a third- party, e.g., "Log in via _Twitter_," "Log in via _Facebook_," "Log in via _Google_," etc.

We don't be supporting "Log in via Twitter" yet, so when you go to create a Twitter application the only fields that matter are the _application name_ (which must be unique across all Twitter applications) and _application description_. The _application URL can be anything_ and the _callback URL can be blank_.

Note: You'll need a _callback URL_ in a world where you want to support "Log in via Twitter."

After creating your application you'll be _redirected_ to your _application configuration page_. The URL should look like
￼￼￼￼￼￼￼￼￼
```
https://dev.twitter.com/apps/<#application ID>/show
```

At the bottom of the page you'll see a section called _Your access token_, which looks roughly like this:

[http://cl.ly/image/340S2F2t0V3Q](http://cl.ly/image/340S2F2t0V3Q) . Create an _access token_ for yourself. You now have _all the information_ you need to _build a Twitter client_. Follow the directions in the _configuration section_ of the Twitter gem.

Here's a simple _test_ of whether you understand how the _Twitter gem and API work_ and whether your _environment is set up correctly_. Can you write a simple _command-line Ruby program_ — no more than 5-10 lines — to _tweet_ something from the _command line_ on your _Twitter account_?

If you want to use _rake console_ you'll have to _require 'twitter'_ and _configure_ the _Twitter gem_ in your _environment.rb_ file. While you can require these _keys_ directly in your _environment file_, this is not a good idea if you are _uploading your applications_ to _GitHub_ or otherwise making this code (and your _keys_) _public_. To avoid this, you can put your keys in a _yaml file_ and load it in your _environment.rb_ file and then put this _yaml_ file in your _.gitignore file_ so you can access it _locally_ but it will not be uploaded to _GitHub_. See [this post](https://gist.github.com/dbc-challenges/c513a933644ed9ba2bc8)


###Recent Tweets (not cached)
Create a routes that looks like this: 
```
get '/' do end
get '/:username' do end
```

Make /:username display the 10 most recent tweets of the supplied Twitter username. Edit
environment.rb to add the appropriate configuration.

Don't worry about leaking your development credentials into the public for now.

###Recent Tweets (cached)
The above URL will be pretty slow. Every time you access it you have to make an API request, which could take a second or more. Let's create a local cache of the results so it's only slow the first time we get a list of recent tweets.
Create models Tweet and TwitterUser. Implement something like the following: 
```

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
