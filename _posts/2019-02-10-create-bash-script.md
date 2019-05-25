---
title: Bash Script
excerpt: Getting to know Bash.
published: true
date: 2019-02-10
categories: project
tags: bash
---
## Basic about Bash Script

More on [here](https://www.linux.com/LEARN/WRITING-SIMPLE-BASH-SCRIPT) and [here](http://matt.might.net/articles/bash-by-example/).

[Bash Guide for Beginners](http://tldp.org/LDP/Bash-Beginners-Guide/html/)

Script starts with a "shebang" for the shell to decide which interpreter to run the script for example bash, tsch, zsh, Perl, Python etc.
``` bash
#!/bin/bash
#!/usr/bin/env python3  #!/path/to/interpreter
```

To  set the script executable:
``` bash
chmod u+x scriptname
```
- `u` - user
- `+` - add permission
- `x` - executable


### Variables
``` bash
#!/bin/bash
# rsync using variables

SOURCEDIR=/home/user/Documents/
DESTDIR=/media/diskid/user_backup/Documents/

rsync -avh --exclude="*.bak" $SOURCEDIR $DESTDIR
```

### Taking input
``` bash
#!/bin/bash

echo $1
```
``` bash
#!/bin/bash

echo -e "Please enter your name: "
read name
echo "Nice to meet you $name"
```

``` bash
#!/bin/bash

echo -e "What directory would you like to back up?" 
read directory

DESTDIR=
 This e-mail address is being protected from spambots. You need JavaScript enabled to view it
 :$directory/

rsync --progress -avze ssh --exclude="*.iso" $directory $DESTDIR
```


## Shortcut command

### Simple alias
``` bash
alias agi='sudo apt-get install'
```
Add line to end of `~/.bashrc` file.

### Start a script
``` bash
#!/bin/bash

echo "changing directory.."
cd ~/project/wayneyee.github.io
echo "starting jekyll.."
bundle exec jekyll serve
```
Change file permission. Add to $HOME/bin.

Add line to end of `~/.bashrc` file.
``` bash
export PATH=$PATH:$HOME/bin
```
- `export` - make variable available for children of this terminal
- `PATH` - work on variable named PATH
- `=` - assign it a new value, which is
- `$PATH` - the old value
- `:` - a separator
- `$HOME/bin` - and your newly created bin directory

More [reading](https://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path).


Start using the new bashrc file:
``` bash
source ~/.bashrc
```
{% gist 1ae4cced70b75b012d1a69c061619fa1 %}

## Useful software
- [Youtube downloader](https://github.com/rg3/youtube-dl/blob/master/README.md#options)
`youtube-dl --write-auto-sub https://www.youtube.com/playlist?list=PLLssT5z_DsK-h9vYZkQkYNWcItqhlRJLN`
- [Kindle-friendly pdf converter](http://www.willus.com/k2pdfopt//)
`k2pdfopt *.pdf -dev kp3`