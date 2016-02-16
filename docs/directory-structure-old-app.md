---

template:      article
title:         Directory structure
naviTitle:     Directory structure - Old App
lead:          The predefined set of folders for each App you need to know for the Old App.
dontList:      true
oldApp:        true
linkOther:     /directory-structure

seeAlsoLinks:
    - deployment
    - domains

keywords:
    - create directory
    - create
    - folder
    - directory
    - directories
    - linux
    - unix

tags:
    - beginner

---

## Your folders

```nohighlight
bin
dev
etc
lib64
proc
tmp                < 1GB temporaray files
usr
var
  www
   web
     appname       < you are here
       data
       htdocs      < default web root
       log
         apache    < web server logs
         php       < runtime logs
       .ssh
```


### appname

The default entry point for [SSH](ssh-sftp-old-app), move downwards and upwards from there on. Please mind that only [Old Apps](new-apps) offer direct file access.

### htdocs

The default web root (aka document root) of your App, everything is online in here. When you create a new App you will find that `index.php` file that prints out the "It works!" message. You can change the [routing point](domains#toc-set-a-custom-root-path), but only below the `htdocs` folder. The [Git deployment](git) syncs to this folder. The `htdocs` folder is part of your App's available storage contingent.

### data

For your internal usage, aka stuff that is offline. Meant to store data, which is not accessible via HTTP, but still permanently stored. The `data` folder is part of your App's available storage contingent.

### log/apache

Apache access and error logs will be written here.

### log/php

PHP error logs and [Worker](workers) output log (worker-out.log).

### tmp

Temporary folder, limited to 1GB of storage. Files older than 15 days will be regularly purged. Typical use cases are the default PHP session file folder, temp destination for file uploads (before `move_uploaded_file()` is called) and a location to download and extract archives in before installing them in your htdocs folder. The tmp folder is available on any Node as well as on SSH/SFTP, however the contents are not shared.

#### Emptying the tmp folder

You cannot access your App's `/tmp` folder directly. If you want to wipe it, you need to write a small script, upload it (temporarily) and execute it directly from your App.

Here is a short example. Save it in `cleanup.php`, upload it and run in once. Then delete it.

```php
foreach (glob("/tmp/*") as $file) {
    unlink($file);
}
```

You can access a `/tmp` folder from your SSH account. However, this `/tmp` folder and your App's are not the same.


## Other folders

The rest of the folders (and files which are not shown here) are part of the standard Linux distribution. All this stuff is handled by us for you. So you don't have access privileges. You can't change things outside the above outlined context.

In most most cases, you probably want to do this to match a certain (framework) directory structure. Just create this structure below `htdocs` and route your domain to the appropriate folder.
