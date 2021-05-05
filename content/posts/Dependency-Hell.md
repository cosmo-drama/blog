+++
title = "Dependency Hell: libc6"
date = "2021-05-05T17:02:31-04:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = "Unheld packages, mismatched versioning, deleted packages and more!"
showFullContent = false
+++

I recently ran into a strange issue when trying to install a package and a few plugins:


```
The following packages have unmet dependencies:
    libicu-dev : 
        Depends: libc6-dev but it is not going to be installed or libc-dev
E: Unable to correct problems, you have held broken packages.
```

Unheld packages? 

```
dpkg --get-selections | grep hold
```

Nothing appeared to be marked 'held', so I went ahead and ran the following commands which usually sets things straight: 

```
sudo apt-get install -f
sudo apt-get clean && sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove

```

After running those, I tried to install `libc6-dev`, the dependency  `libicu-dev` depended on:

```
sudo apt install libc6-dev
```

which resulted in:

```
The following packages have unmet dependencies:
 libc6-dev : Depends: libc6 (= 2.31-0ubuntu9.2) but 2.31-0ubuntu9.3 is to be installed
E: Unable to correct problems, you have held broken packages.
```

Once more, I tried to install `libc6`, the package `libc6-dev` depended on and was met with another frustrating yet promising message:

```
libc6 is already the newest version (2.31-0ubuntu9.3)
```

`libc6` is at the newest version, but `libc6-dev` depends on the older version of `libc6: 2.31-0ubuntu9.2`

To confirm this I ran the following command:

```
apt-cache policy libc6-dev
```

```
libc6-dev:
  Installed: (none)
  Candidate: 2.31-0ubuntu9.2
  Version table:
     2.31-0ubuntu9.2 500
        500 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages
     2.31-0ubuntu9 500
        500 http://us.archive.ubuntu.com/ubuntu focal/main amd64 Packages
```

and then:

```
apt-cache policy libc6
```

```
libc6:
  Installed: 2.31-0ubuntu9.3
  Candidate: 2.31-0ubuntu9.3
  Version table:
 *** 2.31-0ubuntu9.3 100
        100 /var/lib/dpkg/status
     2.31-0ubuntu9.2 500
        500 http://us.archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages
     2.31-0ubuntu9 500
        500 http://us.archive.ubuntu.com/ubuntu focal/main amd64 Packages

```

Apparently, the `2.31-oubuntu9.3` version had recently been deleted as of May 3rd, 2021. 

To fix this, I specified the older version of libc6 to be installed:


>*YMMV: Sometimes reverting packages can break things in unexpected ways, so run this next command at your own risk* 

```
sudo apt-get install libc6=2.31-0ubuntu9.2
```

Once the right version was installed, the rest of the installation process was smooth sailing.

Hope this helps you if you find yourself stuck in some kind of similar layer of dependency hell. 