---

template:   article
title:      Install Slim Framework 2
naviTitle:  Slim Framework 2
lead:       Slim is a PHP micro framework that helps you write simple web applications and APIs quickly. Learn how to install and tune Slim v2 on fortrabbit.

tags:
    - php
    - install

seeAlsoLinks:
    - install-slim-framework-3

---

WORD: This is for Slim in version 2, a guide for the newer [Slim 3](install-slim-3) is also available.

[Slim framework](http://www.slimframework.com/) is easy to setup thanks to Composer. We assume you've already created an [App](app) with fortrabbit. You will also need a local Slim framework installation. You can either use an existing or a new one. This guide explains you how to start locally from scratch:

```bash
# change into your projects directory
$ cd Projects

# clone your fortrabbit app
$ git clone git@deploy.eu2.frbit.com:your-app YourApp

# change into the App directory
$ cd YourApp

# install slim via composer
$ composer require slim/slim "2.*"
```

Now just create your first hello world `index.php` file containing:

```php
<?php

include __DIR__. '/vendor/autoload.php';

$app = new \Slim\Slim();
$app->get('/hello/:name', function ($name) {
    echo "Hello, $name";
});
$app->run();
```

You also want to create an `.htaccess` file, so you have nice URLs:

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```

Create a `.gitignore` file to exclude the `vendor/` folder:

```bash
$ echo vendor/ > .gitignore
```

Now deploy everything and you are done:

```bash
# add everything to git
$ git add -A

# create an initial commit
$ git ci -am 'Initial'

# push and remember the remote
$ git push -u origin master
```

That's it. You can see the hello-world in the browser: `http://your-app.frb.io/hello/world`.
