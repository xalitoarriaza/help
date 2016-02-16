---

template:   article
title:      Install Slim Framework 3
naviTitle:  Slim Framework 3
lead:       Slim is a PHP micro framework that helps you write simple web applications and APIs quickly. Learn how to install and tune Slim v3 on fortrabbit.

tags:
    - php
    - install

seeAlsoLinks:
    - install-slim-framework-2

---


[Slim framework](http://www.slimframework.com/) is extremely easy to setup thanks to our Composer support. We assume you've already created an [App](app) with fortrabbit. You also need a local Slim framework installation. You can either use an existing or a new one. This guide explains you how to start locally from scratch:

```bash
# change into your projects directory
$ cd Projects

# clone your fortrabbit app
$ git clone git@deploy.eu2.frbit.com:your-app YourApp

# change into the App directory
$ cd YourApp

# install slim via composer (currently unstalble, hence the @dev)
$ composer require slim/slim "3.*@dev"
```

Now just create your first hello world `index.php` file containing:

```php
<?php

include __DIR__. '/vendor/autoload.php';

$app = new \Slim\App;
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $response->write("Hello " . $args['name']);
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

