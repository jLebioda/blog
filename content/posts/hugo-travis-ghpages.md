---
title: "Hugo, Travis and GH-pages"
date: 2018-02-14T16:42:11+01:00
draft: false
---

So, you want to integrate _Hugo_ with _Travis CI_ and _Github Pages_? And you want to have the website rebuilt every time you commit to the repository?
You've looked it up on Google, found some tutorials, but yet it does not work as intended? That was my story, too!

I got almost everything correct. To begin with, <!--more--> I followed [this tutorial](https://medium.com/zendesk-engineering/how-to-create-a-website-like-freshswift-net-using-hugo-travis-ci-and-github-pages-67be6f480298), which seems to cover everything. Yet, there were some issues I struggled upon and some errors I made.

 * The theme didn't make it to the git repository. Then, Hugo built only RSS feed.
 
 * I've cloned the theme, instead of adding it as a submodule. Look [here, that it's better to use submodule](https://discourse.gohugo.io/t/adding-a-theme-as-a-submodule-or-clone/8789).
  
 * I've been to `https://travis-ci.com/` (works with private repositories) and not to `https://travis-ci.org/` (works with public repositories). That one was funny, but actually took me around 10 minutes...
 
 * I haven't set up baseURL in `config.toml`. It should be: `baseURL = "https://jlebioda.github.io/blog"`. Without it, all the website's assets just won't work (like stylesheets).

 * The tutorial assumed, that one has _Hugo_ binary in the repository - if you look closely, there's a reference to `binaries/hugo` in `.travis.yml` file. Initially I mistook it for `/binaries/hugo` and thought it's provided by Travis by default (like - `binaries/* == collections of common tools` - I cannot hide that I have never used Travis before). Then I realized, that someone just created a directory called `binaries` and put `hugo` file inside... _Clever, I must admit_ - but pollutes the repository with unnecessary binary file.
 
 Setting it up correctly should be as easy as just downloading the Hugo package during the install step. Therefore, I followed another tutorial which suggested using:
 
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

  This `.travis.yml` file feels a bit overkill (why to set-up whole _go_ environment, if all I need is just a Hugo binary?), _but it works_.
  BTW, stop for a second and think about all the resources wasted by the developers from all over the world. Which just want to use one small binary file. And they create stuffed environments to fetch the file in a convenient way, because that was the way suggested by the tutorial...
  
  
  **EDIT, 2018-02-16**
  
  I couldn't let this wastage (of installing _go_ just to fetch _Hugo_ binary) to stay.
  Additionally, I removed `--theme` switch from `hugo` command - it's redundant, as this information is already in the config.  
  Therefore, my updated `.travis.yml` looks like the following:
  
	language: python
	
    addons:
      apt:
        packages:
          - python-pygments
		  
    install:
      - wget https://github.com/gohugoio/hugo/releases/download/v0.36.1/hugo_0.36.1_Linux-64bit.deb
      - sudo dpkg -i hugo*.deb
      - rm -rf public || exit 0

    script:
      - hugo
	  
    deploy:
      provider: pages
      skip_cleanup: true
      github_token: $GITHUB_TOKEN
      local_dir: public
      on:
        branch: master
	
  Now, it uses `wget` and `dpkg`, which are present even on `python` image in Travis. The major drawback of this approach is being bound to 0.36.1 version. To use newer version, once available, one need to update the `.travis.yml` file. On the other hand, the site builds now under 1 minute (5 times faster than using _go_!) - and that's the reason I'll stay with the _python_ version.
  