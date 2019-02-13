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
In case couldn't run, install libraries.
```
sudo apt-get install libpng-dev
sudo apt-get install --reinstall zlibc zlib1g zlib1g-dev
```



## [Export Jupyter Notebook to Markdown](http://www.leeclemmer.com/2017/07/04/how-to-publish-jupyter-notebooks-to-your-jekyll-static-website.html)

``` bash
jupyter nbconvert --to markdown my_blog/database/insert_sql.ipynb --config jekyll.py
```

{% gist 461d3bcb227cdbc348f463c117707d29 %}