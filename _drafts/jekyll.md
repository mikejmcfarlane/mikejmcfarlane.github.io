# GitHub Pages and Jekyll

- [GitHub Pages and Jekyll](#github-pages-and-jekyll)
  - [Getting started](#getting-started)
  - [Running it locally](#running-it-locally)
  - [SEO](#seo)

## Getting started

* [Creating and Hosting a Personal Site on GitHub](http://jmcglone.com/guides/github-pages/)
* [Jekyll on macOS](https://jekyllrb.com/docs/installation/macos/)

## Docker on Raspberry pi

[How To Install Docker On Raspberry Pi](https://phoenixnap.com/kb/docker-on-raspberry-pi)

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh 
sudo usermod -aG docker pi

<LOGOUT and LOGIN>

docker version
docker info
docker run hello-world

### Running it locally - Mac

* [jekyll/jekyll](https://hub.docker.com/r/jekyll/jekyll)
* [Jekyll Docker](https://github.com/envygeeks/jekyll-docker)

```
cd ~/repos/mikejmcfarlane.github.io
export JEKYLL_VERSION=3.8
docker run --rm --volume="$PWD:/srv/jekyll" -it -p 4000:4000 jekyll/jekyll:$JEKYLL_VERSION jekyll serve
```

* [Then published to localhost](http://localhost:4000)

### Running it locally - Pi

```bash
docker run -d --name jekyll -p 4000:4000 --volume "$PWD:/var/www" ahmetozer/jekyll - NOT WORKING, see dockerfile https://hub.docker.com/r/ahmetozer/jekyll/dockerfile
```

## Running jekyll directly on a Pi

Useful guidance, didn't follow exactly:

+ [How to Install Jekyll on Raspberry Pi 3 (Raspbian Jessie)?](https://stackoverflow.com/questions/38838549/how-to-install-jekyll-on-raspberry-pi-3-raspbian-jessie)
+ [Installing Jekyll on a Raspberry Pi](https://dave.thwaites.org.uk/website/installing-jekyll.html)


```bash
sudo apt install ruby-full
sudo gem install jekyll
sudo gem install bundler
cd repos/mikejmcfarlane.github.io/
bundle install
bundle exec jekyll serve --host 10.0.4.40
```

## SEO

[Verify site and submit sitemap.xml](https://search.google.com/search-console/)
