---

template:      article
naviTitle:     Hello world
title:         Deploy your first code on fortrabbit
lead:          A basic tutorial to help you getting started.
videoid:       47324293849374FFF


keywords:
    - tutorial
    - guide
    - deployment
    - Git
    - terminal
    - shell
    - bash

seeAlsoLinks:
    - dashboard
    - app
    - app-design

tags:
    - beginner
    - install

---


In the following example you'll learn how to use the elementary [Git push](git) to deploy method here on fortrabbit. We assume that you are a bit familiar with the shell, have a [fortrabbit Dashboard](dashboard) Account, already created an [App](app), are aware your Apps Git remote address. The following code gets executed on your local machine:

```bash
# Clone the repo from fortrabbit
# Initialize Git by cloning the empty repo from fortrabbit
$ git clone git@deploy.eu2.frbit.com:my-app.git
Cloning into 'my-app'...
warning: You appear to have cloned an empty repository

# Get in position
# Change directory to your new local project
$ cd my-app

# Make a change
# Create an index php file with a bold messsage on the fly
$ echo '<?php echo "PHPower to the PHPeople";' > index.php

# Thanks for the add
# Add the change in the working directory to the staging area
$ git add index.php

# You did it, commit it
# Commit the staged snapshot to the project history
$ git commit -am 'initial commit'

# Le grand final
# Just push to deploy
$ git push -u origin master
Counting objects: 3, done.
Writing objects: 100% (3/3), 251 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)

Commit received, starting build of branch master

–––––––––––––––––––––––  ∙ƒ  –––––––––––––––––––––––

B U I L D


Checksum:
  3ec951f4f4a59a42afa7e6bebaa78477672d3b6a

Deployment file:
  not found

Pre-script:
  not found
  0ms

Composer:
  not executing (composer.json missing)
  0ms

Post-script:
  not found
  0ms



R E L E A S E


Packaging:
  6ms

Revision:
  1446568383255266100.3ec951f4f4a59a42afa7e6bebaa78477672d3b6a

Size:
  182B

Uploading:
  134ms

Build & release done in 217ms, now queued for final distribution.

–––––––––––––––––––––––  ∙ƒ  –––––––––––––––––––––––

To git@deploy.eu2.frbit.com:my-app.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```