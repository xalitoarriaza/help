---

template:      article
title:         Directory structure
naviTitle:     Directory structure
lead:          The predefined set of folders for each App you need to know.
linkOther:     /directory-structure-old-app

seeAlsoLinks:
    - deeployment
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
tmp               < 2GB temporaray files
usr
srv
  app
    appname
      htdocs      < default web root
```


### appname

The actual name of your App, for example `/srv/app/my-app` and so on.

### htdocs

The default web root (aka document root) of your App. Directory that forms the main directory tree visible from the web. You can change the [routing point](domains#toc-set-a-custom-root-path), to any folder below the `htdocs` directory. The [Git deployment](git) syncs to the `htdocs` folder as well.

### tmp

Temporary folder; limited to 2GB of storage. Files older than 15 days will be automatically purged. Typical use cases are the default PHP session file folder or a temp destination for file uploads via PHP (before `move_uploaded_file()` is called).

For multi node plans: The `tmp` folder is not shared. Meaning that each node has "their own" temporary folder.

## Other folders

The rest of the folders (and files which are not shown here) are part of the standard Linux distribution. All this stuff is handled by us for you. So you don't have access privileges. You can't change things outside the above outlined context.
