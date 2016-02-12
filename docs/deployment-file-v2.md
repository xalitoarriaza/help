---

template:      article
title:         Advanced Git deployment workflows with fortrabbit.yml
naviTitle:     Deployment file
lead:          Enhance your deployment process with the fortrabbit.yml deployment file.
linkOther:     deployment-file-v1-old-app

seeAlsoLinks:
    - git
    - composer

tags:
   - advanced
   - git


---

## Problem

[Git push to deploy](git) is nice and easy to get your code up and running quickly. But deployment is not deployment — there is a huge variety of use cases and preferred workflows. The `fortrabbit.yml` is an optional helper to support your habits. With the deployment file version you can add post- or pre-deploy scripts and control how Composer behaves.

## Solution

Create a file named `fortrabbit.yml` in the App's root folder of your project and include it into Git. After each push, the friendly fortrabbit bots will look for the file and parse it if existent.

## Full schema

```yml
# differentiate from the deployment files for Old Apps
version: 2

# called before Composer runs
pre:
    
    # relative to ~/htdocs
    path: my-script.php

    # optional parameters
    args:
        - foo
        - bar
        - baz

# optional Composer settings
composer:
    
    # Per default dist is prefered
    prefer-source: false

    # Resolves to the --no-dev parameter
    no-dev: false

    # Resolves to the --no-plugins parameter
    no-plugins: false

    # Resolves to the --no-scripts parameter
    no-scripts: false

# called after Composer runs
post:

    # relative to ~/htdocs
    path: sub/folder/my-script.php
    
    # optional parameters
    args:
        - foo
        - bar
        - baz

# list of sustained folders in ~/htdocs. If not given, then it defaults to the "vendor" folder
sustained:
    - vendor

```


## A minimal example

A fortrabbit.yml which calls the PHP script `post.php` on top-level of the repo after Composer looks like this:

```yml
version: 2
post:
    path: post.php

```

## Multi staging use case

To allow using a single git repo with [multiple remotes](multi-staging) you can use an App name based deployment file name.

Assuming you are using two Apps, `my-app-prod` and `my-app-stage`, you can setup two deployment files with named: `fortrabbit.my-app-prod.yml` and `fortrabbit.my-app-stage.yml`. This way, you can have both deployment files in a single repo but which one is to be used is determined by the App you deploy to.