---
title: "New version of Twitter Lib Explained"
author: "Bradley Momberger"
excerpt: "Twitter Lib for Google Apps Script is now at version 16.  If you've created older app scripts based on Twitter Lib version 11 or below, they will not work with the new lib without some conversion.  Version 12 of this library was motivated by the old method of OAuth authentication being officially dropped from Apps Script.  A long-deprecated version was available until recently, which better supported standalone scripts and a simple, more procedural style of Tweeting."
---

# 0: Motivation

[Twitter Lib for Google Apps Script][Twitterlib] is now at version 16.  If you've created older app scripts based on Twitter Lib version 11 or below, they will not work with the new lib without some conversion.

Version 12 of this library was motivated by the old method of OAuth authentication being officially dropped from Apps Script.  A long-deprecated version was available until recently, which better supported standalone scripts and a simple, more procedural style of Tweeting.  Versions 13-16 address bugs that arose after version 12 was deployed.  The most recent version is always recommended, and I try to remember to Tweet every major update.

Google Script Engine is built around two major use cases: building rich functionality for spreadsheets, documents, and forms; and creating Web applications using a standalone script that builds pages and handles GET and POST requests.  Twitter Lib is built around a supported but lesser known use case: running scripts at a regular interval in a standalone container.  Setting up these triggers, or even just running functions directly from the script editor, means that the script writer does not have access to a UI element that may pop up a modal window or otherwise display HTML to a user.

So when authenticating with OAuth, going to an external library is now required.  Google has helpfully prepared this library for public use, but at the same time, it's clunkier than the old style and requires a few extra steps.

## 0.1: Review of Twitter Lib

Twitter Lib is meant to be the most simple interface possible to the most common Twitter operations for bots: searching, Tweeting, retweeting, favoriting, and media upload.  The target user of Twitter Lib is either a programmer with any amount of experience writing code, or someone newish to programming who wants to try writing a Twitter bot with script for the increased flexibility over a Web- or spreadsheet-based bot builder.  Most Twitter automation involves a wrapper around the call to the GET or POST operation to the API, taking care to avoid gotchas and providing a garden path to a successful operation.

# 1: The `Twitterlib.OAuth` object

Previously the use of Twitter Lib was procedural.  The user would call, for example, `Twitterlib.oAuth()` and then `Twitterlib.fetchTweets()`.  This is no longer possible with the new OAuth library -- it has to manage things a little differently because it doesn't have preferential treatment from Script Engine.

Because the OAuth1 library provided by Google uses an object as its main mode of operation, it made sense to closely mirror how OAuth1 is implemented.  Strictly speaking, Twitter Lib now descends from OAuth1 in the prototype chain; loosely speaking, Twitter Lib subclasses OAuth1 but tries to be easier to use.

Where before you did this:

```javascript
var props = PropertiesService.getScriptProperties();
Twitterlib.oAuth(
    props.getProperty("TWITTER_CONSUMER_KEY"),
    props.getProperty("TWITTER_CONSUMER_SECRET")
);
Twitterlib.fetchTweets('"in case you missed it" OR icymi');
```

Now you do this:

```javascript
var props = PropertiesService.getScriptProperties();
var twit = new Twitterlib.OAuth(props)
               .setConsumerKey(props.getProperty("TWITTER_CONSUMER_KEY"))
               .setConsumerSecret(props.getProperty("TWITTER_CONSUMER_SECRET"))
               .setAccessToken(props.getProperty("TWITTER_ACCESS_TOKEN"))
               .setAccessTokenSecret(props.getProperty("TWITTER_ACCESS_SECRET"));
twit.fetchTweets('"in case you missed it" OR icymi');
```

This seems like a lot of extra lines to do the same thing, what with configuring that `twit` object, but I've tried to help this along and make this simpler by reading the property store for common things.

# 2: Use of stored properties

For convenience, the consumer keys and access tokens do not need to be explicitly set if they're already in the property store.  The table below shows the necessary keys to store those values under:

|item to store|key to use in properties|
|-------------|----------|
| consumer key | TWITTER\_CONSUMER\_KEY |
| consumer secret | TWITTER\_CONSUMER\_SECRET |
| access token | TWITTER\_ACCESS\_TOKEN |
| access token secret | TWITTER\_ACCESS\_SECRET |

Since we've used these exact key names in our code above, the four lines where we set them aren't necessary, and we can just do this:

```javascript
var props = PropertiesService.getScriptProperties();
var twit = new Twitterlib.OAuth(props);
twit.fetchTweets('"in case you missed it" OR icymi');
```

Now it's looking as simple as the old style!

# 3: Options for authorizing

One of the important things I'm attempting to support in Twitter Lib is the different use cases that might crop up when using the library in different contexts.  Previously, in old versions of the lib, it was never necessary to set the access token and access token secret.  Those were stored when the bot writer started the workflow manually and clicked through the OAuth authorization popup.  This isn't available from the script editor any longer, so there are new options for getting that info into the script.

> Note: until an access token and access token secret are being passed to the `Twitterlib.OAuth` instance, all of the functions that connect to the Twitter API (`fetchTweets()`, `sendTweet()`, `retweet()`, etc.) will throw errors.  It is also not possible to do the two-step authorization flows below in one step, since the popup window or emailed link goes outside of the script context.

## 3.1: One-step authorization.

If you generate your Twitter access token along with your API keys from [https://apps.twitter.com/](https://apps.twitter.com/), there's no need to go through a flow of authorization to be authorized.  Just set all four of the values in the table in Section 2 when making an instance of `Twitterlib.OAuth` and you'll be set to go.  For most bot makers this is the preferred way, and more closely mirrors how users are setting up Twitter API access from Node or Python.

```javascript
var props = PropertiesService.getScriptProperties();
props.setProperty("TWITTER_CONSUMER_KEY", "<consumer key>");
props.setProperty("TWITTER_CONSUMER_SECRET", "<consumer secret>");
props.setProperty("TWITTER_ACCESS_TOKEN", "<access token>");
props.setProperty("TWITTER_ACCESS_SECRET", "<access token secret>");
var twit = new Twitterlib.OAuth(props);
```

## 3.2: Authenticating from with a document, spreadsheet, or form.

If you opt not to go the one-step route (if for example, you want each user to run the same script in different contexts, or just to set up one Twitter app so that your bot generator users don't have to make apps themselves) and you're setting up the script to run in a Google document instead of standalong, you can instead only set the consumer key and secret, then when it comes time to authorize, a popup will be automagically presented to the user with the Twitter authorization flow.  This only needs to be done the first time.

```javascript
//some of the code below adapted from Zach Whalen's bot spreadsheet
function doAuthorization() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName('Setup');
  var consumer_key = sheet.getRange('b23').getValue();
  var consumer_secret = sheet.getRange('b26').getValue();

  var service = new Twitterlib.OAuth(PropertyService.getScriptProperties)
                .setConsumerKey(consumer_key)
                .setConsumerSecret(consumer_secret);
  if (!service.hasAccess()) {
    service.runAuthorizeFlow();
  }
}

function authCallback(request) {
  var service = new Twitterlib.OAuth(PropertiesService.getScriptProperties());
  service.handleCallback(request);
}
```

The name "authCallback" for the callback function is a default.  If you need to change the name for any reason, call `.setCallbackFunction(<function name>)` on the service in `doAuthorization()`.

## 3.3 Authorization over email

If you have a standalone script and don't have access tokens for any reason, you can still get the tokens through almost the same flow as above, but with one key difference.  Instead of getting a popup window when running `runAuthorizeFlow()`, you'll receive an email at your GMail address with a link to the authorization page on twitter.com.  You will still need to set properties, consumer key, and consumer secret.

# 4: API Reference

## 4.1: `Twitterlib.OAuth()` constructor

Create a new OAuth object with `new Twitterlib.OAuth()`.  Since a property store must be set before the OAuth object can be used, the constructor takes the property store as an optional argument.

## 4.2: Setters

Refer to [the OAuth1 documentation][OAuth1] for setters not described here.  Note that the access/request/authorize values are automatically assigned by Twitter Lib, as is the authorization callback function.

### 4.2.1: `setAccessToken(token)`

`token` is a string representing the access token for the current user and API app.

Returns the OAuth object for chaining.  If the property store is set, then this will also set the key "oauth1.twitter" on the property store

### 4.2.2: `setAccessTokenSecret(token)`

`token` is a string representing the access token secret for the current user and API app.

Returns the OAuth object for chaining.  If the property store is set, then this will also set the key "oauth1.twitter" on the property store

## 4.3: `runAuthorizeFlow()`

Runs the appropriate authorization flow for the script context (popup window for a document, spreadsheet, or form; email for standalone).

Returns nothing.

## 4.4 `checkAccess()`

Throws an error if no access token is found in the property store.  

Returns nothing.  

This is primarily used by API-connecting functions to avoid messy errors if OAuth authorization hasn't been completed.

## 4.5 `sendTweet(tweet, params)`

`tweet` is a Tweet object with a `text` property, or a string

`params` is an optional object of any additional params, e.g. `media_ids`

Throws an error if no access token is found in the property store.  

Returns an object of the JSON response from Twitter's service if successful, false if unsuccessful.

Posts a status update to Twitter with the appropriate text.  Logs response to the Logger on success and failure.

## 4.6 `favorite(tweet)`

`tweet` is a Tweet object with an `id_str` property, or a string representing the Tweet's ID

Throws an error if no access token is found in the property store.  

Returns an object of the JSON response from Twitter's service if successful, false if unsuccessful.

Favorites a Tweet already posted to Twitter.  Logs response to the Logger on success and failure.

## 4.7 `retweet(tweet)`

`tweet` is a Tweet object with an `id_str` property, or a string representing the Tweet's ID

Throws an error if no access token is found in the property store.  

Returns an object of the JSON response from Twitter's service if successful, false if unsuccessful.

Retweets a Tweet already posted to Twitter.  Logs response to the Logger on success and failure.

## 4.8 `fetchTweets(search, tweet_processor, options)`

`search` is a string in Twitter's query language ([See here for more](https://dev.twitter.com/rest/public/search))

`tweet_processor` is an optional function that takes each tweet retrieved from the API.  If the function returns a falsy value, the tweet is not returned from `fetch_tweets()`.  If the tweet returns `true`, the original tweet is returned.  If the tweet returns a string or an object, that string or object will be returned from `fetch_tweets()`.

`options` is an optional object with the following possible keys:

* `multi`: set to `true` to return an array of tweets instead of one tweet; all tweets will be run through the processor function and only be included if the function returns truthy values.

* `count`: set to a number to fetch that number of tweets from the API before processing (default: 5)

* `lang`: set to an ISO language code to change the language of tweets being searched (default "en")

* `since_id`: set to a Tweet's `id_str` property to only search for Tweets posted after that Tweet.

Throws an error if no access token is found in the property store, or if an error is encountered when accessing the Twitter API.  

Returns a Tweet object matching the search query, or an array of matching Tweets if `multi` is set to `true`.  Returns null if no tweets were found and `multi` is set to `false`.

Searches for Tweets on Twitter.  Logs response to the Logger on failure.

## 4.9 `uploadMedia(blob)`

`blob` is a `Blob` object with the MIME type either set or 

Throws an error if no access token is found in the property store.

returns the Twitter response containing the `media_id` key on success, or null if an error or failure is encountered.

## 4.10 static functions not part of `OAuth` object

### 4.10.1 `encodeString(string)`

`string` is any string

Returns a string.

Does OAuth-compatible percent encoding of the supplied string, which requires a few more characters be escaped than the native JS function `encodeURIComponent()` provides.

### 4.10.2 `grabImage(url, mime_type)`

`url` is a string representing the URL of the image to load into a `Blob` object

`mime_type` is an optional string, representing the MIME type of the image to set in the `Blob`

returns a `Blob` containing the image data.

# 5 Any Questions?

Tweet them [@air_hadoken](https://twitter.com/air_hadoken)

Also, please peruse [the source][Twitterlib] to learn more about how it works.  There is an unfortunate sequencing problem with the fact that I had to include all of OAuth1 in the project, but just look for twitter.gs in the project.

[OAuth1]:https://github.com/googlesamples/apps-script-oauth1
[Twitterlib]:https://script.google.com/d/11dB74uW9VLpgvy1Ax3eBZ8J7as0ZrGtx4BPw7RKK-JQXyAJHBx98pY-7/edit?usp=sharing