---

template:   article
title:      Install Laravel 5 - for Old App
naviTitle:  Laravel 5, Old App
lead:       Deploy Laravel 5 with Composer to fortrabbit.
dontList:   true
oldApp:     true
linkOther:  /install-laravel-5
tags:
    - php
    - install
    - laravel

seeAlsoLinks:
    - install-laravel-4-old-app

---

We assume you have [Laravel](http://laravel.com/docs/5.0/installation) installed with [Composer](https://getcomposer.org/doc/00-intro.md) locally. Also you should have set up a plain vanilla new fortrabbit App on fortrabbit.

Create the project
------------------

Start out with creating a Laravel 5 project via composer, somewhere on your local system:

``` bash
cd ~/your-projects
composer create-project laravel/laravel --prefer-dist MyApp
```

This will take some time and look something like the following:

``` bash
Installing laravel/laravel (v5.0.1)
  - Installing laravel/laravel (v5.0.1)
    Downloading: 100%

Created project in MyApp
Loading composer repositories with package information
Installing dependencies (including require-dev)
  - Installing vlucas/phpdotenv (v1.1.0)
    Loading from cache

# and so on ..

Generating autoload files
Generating optimized class loader
Compiling common classes
Compiling views
Application key [asdasdasdasd] set successfully.
```

Document root configuration
---------------------------

While the above project is locally created, you should change the document root of your App. In the Dashboard you can configure the App to [route all requests](domains#toc-set-a-custom-root-path) to Laravel's `public` folder, which Laravel requires.

Initialize the Git repository
-----------------------------

When the project creation is done change in your local directory and connect your new Laravel 5 project with your App's repo on fortrabbit:

``` bash
cd MyApp
git init .
git remote add origin git@eu1.frbit.com:my-app.git
```

Since the Laravel comes with a good `.gitignore` file you can now safely add all files and create an initial commit with the [fortrabbit composer trigger](#..). Once that's done, just push to deploy.

``` bash
git add -A
git commit -am 'Initial [trigger:composer:install]'
git push -u origin master
```

This can take some time and you will see a lot of `Installing symfony/translation [v2.6.4]` messages. Once it's done you're all set and can visit your new Laravel App in the browser!



