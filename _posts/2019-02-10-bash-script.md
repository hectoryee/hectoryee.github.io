---
title: Bash Script
excerpt: Getting to know Bash. Write and run a bash script.
published: true
date: 2019-02-10
categories: project
tags: bash
---

## 1. Shortcut command
### 1.1. Simple alias
``` bash
alias agi='sudo apt-get install'
```
Add line to end of `~/.bashrc` file.


### 1.2. Start a script
``` bash
#!/bin/bash
echo "changing directory.."
cd ~/project/wayneyee.github.io
echo "starting jekyll.."
bundle exec jekyll serve
```

Add to `$home/bin`.

Make file executable `sudo chmod +x filename`.

Add line to end of `~/.bashrc` file.
``` bash
export PATH=$PATH:$HOME/bin
```
- `export` make variable available for children of this terminal
- `PATH` work on variable named PATH
- `=` assign it a new value, which is
- `$PATH` the old value
- `:` a separator
- `$HOME/bin` and your newly created bin directory

Start using the new bashrc file:
``` bash
source ~/.bashrc
```

### 1.3. Examples
Add to `$home/bin`.

{% gist 1ae4cced70b75b012d1a69c061619fa1 %}



## 2. Write to File
``` bash
# overwrite
output > filename 
# append
output >> filename 
```

``` bash
echo "this is a line" > file.txt
echo "this is a line" | tee file.txt
echo "this is a line" | tee -a file.txt

cat << EOF > file.txt
The current working directory is: $PWD
EOF
```


## 3. Bash Script Basics
[Bash Guide for Beginners](http://tldp.org/LDP/Bash-Beginners-Guide/html/)

Script starts with a "shebang" for the shell to decide which interpreter to run the script for example bash, tsch, zsh, Perl, Python etc.
``` bash
#!/bin/bash
#!/usr/bin/env python3  #!/path/to/interpreter
```

To  set the script executable run `chmod +x scriptname`.
- `+` add permission
- `x` executable


### 3.1. Variables
``` bash
#!/bin/bash
SOURCEDIR=/home/user/Documents/
echo "path" $SOURCEDIR
```


### 3.2. Taking input
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