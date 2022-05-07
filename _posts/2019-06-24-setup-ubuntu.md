---
title: Setup Ubuntu after Booting
excerpt: Guide for setting up dual-boot ubuntu. Include partition size, post install script, software settings and configure Windows to mount and eject drive.
published: true
date: 2019-06-24
categories: project
tags: bash linux
---

## Configuring Windows

### Colours
annotation highlight color - `#b3ffdb6e`


### Foobar
    [%disc number%.][$num(%tracknumber%,2). ]$trim(%title%)[ - %track artist%]


### Mountvol
```
# list volumes
mountvol

# eject
mountvol D: /p

# mount {e864ee90-ea3d-43d9-962d-3452d830dd69}
mountvol D: \\?\Volume{e864ee90-ea3d-43d9-962d-3452d830dd69}\
```


### Disable autorun
1. Windows key + R -> regedit
2. HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies
3. Create a new Dword type value with name "NoDriveTypeAutorun". Set value to "FF".


## Installing Ubuntu
### Disk Partition Size

``` bash
+---------+-----------+----------+
|  DISK   | PARTITION |   SIZE   |
|---------+-----------+----------|
|   SSD   |   /       |   60 Gb  |
|         |   swap    |   12 Gb  |
|   HDD   |   /home   |   60 Gb  |
+---------+-----------+----------+
```

Change ownership of all files run this:

``` bash
sudo chown -R hsunwei /home/hsunwei
```


### Chinese input
1. System Settings–>Language Support–>Install/Remove Languages
2. `sudo apt-get install ibus ibus-clutter ibus-gtk ibus-gtk3 ibus-qt4`
3. `sudo apt-get install ibus-pinyin`
4. Logout
5. sudo ibus-setup


### Shortcut
Bash script, save to `$home/bin`.

{% gist https://gist.github.com/xunweiyee/1ae4cced70b75b012d1a69c061619fa1 %}


### Useful software
- [Youtube downloader](https://github.com/rg3/youtube-dl/blob/master/README.md#options)
`youtube-dl --write-auto-sub https://www.youtube.com/playlist?list=PLLssT5z_DsK-h9vYZkQkYNWcItqhlRJLN`

- [Kindle-friendly pdf converter](http://www.willus.com/k2pdfopt//)
`k2pdfopt *.pdf -dev kp3`

- [HTML 5 outliner for Chrome](https://h5o.github.io/) [HeadingsMap for Firefox](https://addons.mozilla.org/en-US/firefox/addon/headingsmap/) [How can I add a table of contents to an ipython notebook?](https://stackoverflow.com/questions/21151450/how-can-i-add-a-table-of-contents-to-an-ipython-notebook)

- [WakaTime extension for chrome (jupyter notebookls)](https://chrome.google.com/webstore/detail/wakatime/jnbbnacmeggbgdjgaoojpmhdlkkpblgi?hl=en)


### Setup script

{% gist 4ca47b320b0f7e13a16958210eef034c %}
