---
title: Using Jekyll
excerpt: How I use Jekyll.
published: true
date: 2019-02-09
categories: project
tags: jekyll
---
In this post I will record how I have been using jekyll.

## Setting up local Jekyll

Follow [this tutorial](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/) to set up Jekyll.

Install Ruby. Recommended to use [rbenv](https://github.com/rbenv/rbenv).

Install Bundler:
``` bash
gem install bundler
```

Install Jektll and dependencies from [GitHub Pages gem](https://pages.github.com/versions/):
``` bash
bundle install
```

Run Jekyll site locally:
``` bash
bundle exec jekyll serve
```



## Jupyter notebook to blog