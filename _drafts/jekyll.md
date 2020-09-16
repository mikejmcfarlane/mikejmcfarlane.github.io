# GitHub Pages and Jekyll

- [GitHub Pages and Jekyll](#github-pages-and-jekyll)
  - [Getting started](#getting-started)
  - [Running it locally](#running-it-locally)
  - [SEO](#seo)

## Getting started

* [Creating and Hosting a Personal Site on GitHub](http://jmcglone.com/guides/github-pages/)
* [Jekyll on macOS](https://jekyllrb.com/docs/installation/macos/)

## Running it locally

* [jekyll/jekyll](https://hub.docker.com/r/jekyll/jekyll)
* [Jekyll Docker](https://github.com/envygeeks/jekyll-docker)

```
export JEKYLL_VERSION=3.8
docker run --rm --volume="$PWD:/srv/jekyll" -it -p 4000:4000 jekyll/jekyll:$JEKYLL_VERSION jekyll serve
```

* [Then published to localhost](http://localhost:4000)

## SEO

[Verify site and submit sitemap.xml](https://search.google.com/search-console/)
