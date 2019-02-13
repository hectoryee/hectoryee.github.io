---
title: Jupyter Notebook
excerpt: My way of using Jupyter Notebook
published: true
date: 2019-02-11
categories: project
tags: notebook
---

## [Install packages from Jupyter Notebook](https://jakevdp.github.io/blog/2017/12/05/installing-python-packages-from-jupyter/)
``` bash
# Install a pip package in the current Jupyter kernel
import sys
!{sys.executable} -m pip install numpy
```