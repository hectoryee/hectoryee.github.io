---
title: Backup Ubuntu
excerpt: Backing up my OS.
published: true
date: 2019-02-10
categories: project
tags: bash
---
Based on this [post](https://ubuntuforums.org/showthread.php?t=35087).

## Backing-up
Navigate to where to save file.
``` bash
cd /media/home
```
Full command to make a backup of my system.
``` bash
tar cvpzf backup.tgz --exclude=/proc --exclude=/lost+found --exclude=/backup.tgz --exclude=/mnt --exclude=/sys --exclude=/home/hectoryee/Music --exclude=/home/hectoryee/Movies --exlude=/media/home/Music --exclude=/media/home/Movies /
```
- `/` - everything

## Restoring
``` bash
tar xvpfz backup.tgz -C /
```
- `C` - change directory

Will overwrite everything on the partition.