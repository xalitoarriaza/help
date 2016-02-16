---

template:    article
title:       Git deployment
naviTitle:   Git deployment
lead:        Learn how to deploy with Git and see advanced configuration.
dontList:    true
oldApp:      true
linkOther:   /git

keywords:
    - beginner
    - dashboard

tags:
     - git

seeAlsoLinks:
     - git-submodules
     - deployment-file-v1-old-app
     - directory-structure
     - ssh-sftp-old-app
     - security
     - quirks
     - ssh-keys


---

We assume that you have: [Git installed](http://git-scm.com/) locally; your [local SSH keys in place](https://help.github.com/articles/generating-ssh-keys/) and [your public SSH key saved with your fortrabbit Account](ssh-keys). Authentication to access code is done with your public SSH keys.

## Usage

Each fortrabbit App comes with its very own custom Git repo. Register that as a remote origin to your local Git working copy. Then just push your code to that remote Master branch.


### Simplified deployment workflow log

```bash
# Cloning the (empty) app to register the remote origin master
git clone git@git1.eu1.frbit.com:app-name.git

# do stuff
edit index.php

# push to deploy
git add index.php
git commit -am 'Some changes'
git push -u origin master

# et voilà — worship your work in the browser
```

### Behind the scenes

```nohighlight
+----------------+   +----------------+   +-----+
| Your local Git +---> fortrabbit Git +---> App |
+----------------+   +----------------+   +-----+
```

Your `git push` updates the Git remote repo. But that's not the code you see executed in the browser. After the fortrabbit Git remote repo got updated, it gets packed and synchronized into the actual Node containing your App (read webspace).


### Overwrite but not delete default for Old Apps

For [Old Apps](new-apps): The old files will be overwritten with the ones that have been updated thru Git. However, Git deployment will not delete anything. So when you delete a file from your Git repo it will be still there in the webspace. This is the safe default to deal with "runtime contents". When you implement something like an image upload form, you will upload additional contents to your webspace. Those files are generated on the webspace directly. So they are not part of Git repo. But most likely you don't want these to be deleted whenever you push code changes. You can change this in the [deployment file](#toc-deployment-file).

### Full sync strategy for New Apps

For [New Apps](new-apps): All files in the webspace will be replaced with the current release package.


### The master branch counts

While you can have as many Git branches you want, only changes to the master branch will be synchronized into the webspace. Currently there is no way to change which branch shall be deployed to an App. However you can match your local branches to master branches of different Apps to create a [development/production setup](multi-staging). [New Apps](new-apps) also accept and prefer a branch called like the App name.


## Deployment file

Fine tune deployment configurations with the `fortrabbit.yml` deployment file: control the way Composer runs, define pre- & post-deploy scripts and more.

* [deployment file Version 1](deployment-file-v1-old-app) for Old Apps
* [deployment file Version 2](deployment-file-v2) for New Apps


### Composer

New Apps will always run an [Composer](composer) install after a successful `git push`. With the [Old Apps](new-apps) you can use a special formated commit message to [trigger Composer](composer) `install` or `update` on the fortrabbit remote after you pushed some code.


### Private Git/Composer repos

You can also include your own private Composer repository as described [here](private-composer-repos).


### Large files

Don't put big binary files (>2 MB) in Git. This bloats your repository and makes everything slow. Upload them once and add them to your `.gitignore` file. Even better: Structure your contents in a way that you can ignore whole folders containing those large files.


## Integrating with GitHub & Bitbucket

We don't have any fancy GitHub/Bitbucket integrations (yet), but it is easily possible to combine your fortrabbit repo with Bitbucket/Guthub as described [here](bitbucket-github-and-fortrabbit).

