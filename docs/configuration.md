---
title: "Configuration"
---

This gem will work well out of the box, but is configurable in a number of ways. Note: all of these configuration options should nest under a `webmentions` key in your `_config.yml` file.

* `username` - Your [webmention.io](https://webmention.io) username (for use in the `link` tags in your head)
* `templates` - If you would like to roll your own templates, you totally can. You will need to assign a hash of the template paths to use for loading each one.
* `syndication` - Configuration settings which control [syndication](/jekyll-webmention_io/syndication) functionality of the gem
* `bad_uri_policy` - In order to reduce unnecessary requests to servers that aren’t responding, this gem will keep track of them and avoid making new requests to them based on a policy you set.  See [Bad URI Policy](/jekyll-webmention_io/bad_uri_policy) for more details.
* `cache_bad_uris_for` - This setting provides a short-hand method for controlling basic URL error handling by allowing you to set the number of days you want to wait before trying a given domain again.  This behaviour can also be more finely controlled through the Bad URI Policy.
* `max_attempts` - The bad_uri_policy settings control the behaviour of this gem for whole hosts.  This setting allows the user to specify a maximum number of attempts to send a specific webmention before the plugin gives up.  By default this setting is disabled, meaning there is no maximum.
* `cache_folder` - by default, this gem will cache all files in the `.jekyll-cache`, but you can specify another location (like `_data`) if you like. In order to avoid collisions, all cache files will be prefixed with “webmention_io_” unless your `cache_folder` value contains “webmention” (e.g. `.jekyll_cache/webmentions`)
* `html_proofer` - If you use the HTML Proofer gem to check your HTML, it doesn't ignore template tags, so we add the `data-proofer-ignore` attribute to the template elements to avoid showing false positives.
* `legacy_domains` - If you’ve relocated your site from another URL or moved from to HTTPS from HTTP, you can use this configuration option to specify additional domains to append your `page.url` to. It expects an array.


## Simple Example

```yml
webmentions:
  username: YOUR_USERNAME
  # Use my own cache folder
  cache_folder: .cache
  # skip bad URLs for 5 days
  cache_bad_uris_for: 5
  # I moved to www and then to https, so…
  legacy_domains:
    - http://aaron-gustafson.com
    - http://www.aaron-gustafson.com
```

## Exhaustive Example

```yml
webmentions:
  username: YOUR_USERNAME
  cache_folder: .cache
  legacy_domains:
    - http://aaron-gustafson.com
    - http://www.aaron-gustafson.com
  templates:
    count: _includes/webmentions/count.html
    likes: _includes/webmentions/likes.html
    links: _includes/webmentions/links.html
    posts: _includes/webmentions/posts.html
    replies: _includes/webmentions/replies.html
    reposts: _includes/webmentions/reposts.html
    webmentions: _includes/webmentions/webmentions.html
  max_attempts: 5
  bad_uri_policy:
    unsupported:
      policy: ban
    error:
      policy: ignore
    failure:
      policy: retry
      retry_delay: [ 1, 12, 48, 120 ]
      max_retries: 5
    whitelist:
      - "^https://brid.gy/publish/"
    blacklist:
      - "^https://foo.bar/"
  syndication:
    mastodon: 
      endpoint: https://brid.gy/publish/mastodon
      response_mapping:
        syndication: $.url
    github: 
      endpoint: https://brid.gy/publish/github
collections:
  posts:
    syndicate_to: [ mastodon, github ]
```

## What’s checked

Given the time-consuming nature of collecting and sending webmentions, by default this plugin will only look at your site’s posts (stuff in the `_posts` directory or what’s returned in `site.posts.docs`). In reality, there can be many more content types in a Jekyll site and you may want to include them with outgoing webmentions or look for inbound webmentions to them. You can customize your installation to include arbitrary pages and/or Jekyll Collections by altering your `_config.yml`.

Add pages by setting `pages` to `true`:

```yml
webmentions:
  pages: true
```

You can add **all** collections by setting `collections` to `true`:

```yml
webmentions:
  collections: true
```

Since you can have multiple collections and you may not want to apply the same logic to all of them, you can cherry-pick specific collections you’d like to include by setting the `collections` value to an array:

```yml
webmentions:
  collections:
    - links
```

## Picking Up Redirects

If you’ve ever changed the path to your posts, you may have used [the `jekyll-redirect-from` gem](https://github.com/jekyll/jekyll-redirect-from). `jekyll-webmention_io` will look for a `redirect_from` key in your YAML front matter and automatically include that original URL in any requests for webmentions so none get left behind.
