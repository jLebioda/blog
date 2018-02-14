---
title: "Hugo, Travis and GH-pages"
date: 2018-02-14T16:42:11+01:00
draft: false
---

So, you want to integrate _Hugo_ with _Travis CI_ and _Github Pages_? And you want to have the website rebuilt every time you commit to the repository?
You've looked it up on Google, found some tutorials, but yet it does not work as intended?

I got almost everything correct. To begin with, <!--more--> I followed [this tutorial](https://medium.com/zendesk-engineering/how-to-create-a-website-like-freshswift-net-using-hugo-travis-ci-and-github-pages-67be6f480298), which seems to cover everything. Yet, there are some issues and I managed to make some errors:

 * I've cloned the theme, instead of adding it as a submodule. See [that it's better to use submodule](https://discourse.gohugo.io/t/adding-a-theme-as-a-submodule-or-clone/8789).
  
 * Going to `https://travis-ci.com/` (works with private repositories) and not to `https://travis-ci.org/` (works with public repositories). That one was funny, but actually took me around 10 minutes...
 
 * Setting up proper baseURL in `config.toml`: `baseURL = "https://jlebioda.github.io/blog"`. Without it, all the website's assets just won't work (like stylesheets).

 * Having _Hugo_ binary in the repository - if you look closely, there's a reference to `binaries/hugo` in `.travis.yml` file. I cannot hide I haven't used Travis before - initially I mistook it for `/binaries/hugo` and thought it's provided by Travis by default (like - `binaries/* == collections of common tools`). Then I realized, that someone just created a directory called `binaries` and put `hugo` file inside... 
 
 In my opinion, it should be as easy as just downloading the Hugo package during the install step. 
 Therefore, I followed another tutorial which suggested using:
 
  ```
    language: go

    go:
      - master

    addons:
      apt:
        packages:
          - python-pygments
    
    install:
      - go get github.com/spf13/hugo
      - rm -rf public || exit 0

    script:
      - hugo --theme=hemingway
  ```

  It feels a little bit like overkill (why to set-up _go_ environment, if all I need is Hugo binary?), _but it works_.
  BTW, stop for a second and imagine all the resources wasted by the developers from all over the world to solve this type of problems...