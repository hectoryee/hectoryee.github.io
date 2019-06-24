---
title: Setting Up My Ubuntu
excerpt: Post install script, software settings.
published: true
date: 2019-06-24
categories: project
tags: bash
---

## Installing Ubuntu

### Disk Partition Size

``` bash
+---------+-----------+----------+
|  DISK   | PARTITION |   SIZE   |
|---------+-----------+----------|
|   SSD   |     /     |   40 Gb  |
|         |    swap   |   12 Gb  |
|   HDD   |   /home   |   15 Gb  |
+---------+-----------+----------+
```

### Post install script

{% gist 4ca47b320b0f7e13a16958210eef034c %}



Regain ownership of all files run this:

``` bash
sudo chown -R hsunwei /home/hsunwei
```




### Useful software
- [Youtube downloader](https://github.com/rg3/youtube-dl/blob/master/README.md#options)
`youtube-dl --write-auto-sub https://www.youtube.com/playlist?list=PLLssT5z_DsK-h9vYZkQkYNWcItqhlRJLN`
- [Kindle-friendly pdf converter](http://www.willus.com/k2pdfopt//)
`k2pdfopt *.pdf -dev kp3`


## Installing Windows

### Foobar
- [https://www.foobar2000.org/encoderpack]
- foo_discogs
- foo_dsd_processor
- foo_input_dts
- foo_dsp_xgeg
- foo_input_monkey
- foo_input_sacd
- foo_out_upnp
- foo_upnp