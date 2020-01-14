# A blog with jekyll and github pages

There's a few tutorials around the internet already about this, but here's some issue I ran into along to way.

The following errors started when I switched from the minima layout to one of the [Github supported layouts](https://pages.github.com/themes/)

If you get this error:
```
ERROR: YOUR SITE COULD NOT BE BUILT:
                    ------------------------------------
                    No repo name found. Specify using PAGES_REPO_NWO environment variables, 'repository' in your configuration, or set up an 'origin' git remote pointing to your github.com repository.
```

[This comment](https://github.com/jekyll/jekyll/issues/4705#issuecomment-200991736) fixed it for me.

Next error was:

```
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
   GitHub Metadata: Error processing value 'repo_info':
  Liquid Exception: uninitialized constant Faraday::Error::ConnectionFailed Did you mean? Faraday::ConnectionFailed in /_layouts/default.html
```

Seems `GitHub Metadata: Error processing value 'repo_info':` was the fun part, and [this comment](https://github.com/github/pages-gem/issues/399#issuecomment-301827749) solved it for me.

Finally I got this error:

```
Invalid theme folder: _includes
Invalid theme folder: _includes
      [...]
      Generating...
       Jekyll Feed: Generating feed for posts
     Build Warning: Layout 'page' requested in about.markdown does not exist.
     Build Warning: Layout 'home' requested in index.markdown does not exist.
```

Seems switching to the hacker theme made the index.html page stop rendering locally, and I finally gave up and switched back to minima. This is a recreational blog so there's only so much fighting I'm willing to put up with.

At some point I do want this site dark though so I may have to come back to this in the future.
