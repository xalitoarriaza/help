---

template:   article
title:      Install Laravel 4 - for Old App
naviTitle:  Laravel 4, Old Apps
lead:       Deploy Laravel 4 with Composer to fortrabbit.
dontList:   true
oldApp:     true
linkOther:  /install-laravel

tags:
    - php
    - install
    - deploy
    - laravel

seeAlsoLinks:
    - install-laravel-5-old-app
    - app
    - deployment-architecture-video

externalLinks:
    - "asaff sdfsdfdsf:http://blog.fortrabbit.com/10-pillars-php-dev"

---

We assume you have [Laravel](http://laravel.com/docs/4.2/installation) installed with [Composer](https://getcomposer.org/doc/00-intro.md) locally. Also you should have set up a plain vanilla new fortrabbit App on fortrabbit.


## Initial setup with Composer

If you haven't done this already, add your fortrabbit Apps Git repository URL as a remote:

```bash
# use your own App name
# using the fortrabbit remote as master is optional
git remote add origin git@git1.eu1.frbit.com:my-app.git
```

Add everything, make a commit with the additional `[trigger:composer]` keyword in the message and push to the fortrabbit remote. You need to wait a bit, because composer will install quite a lot of packages.

```bash
git add -A
git commit -am 'Laravel 4 install [trigger:composer]'
git push -u origin master
# long output follows here
```
That's basically deploying. The next runs will be much faster.

## Root path configuration

Laravel is installed into the `htdocs` [folder](directory-structure) of your App. In the Dashboard you tell the App to [route all requests](domains#toc-set-a-custom-root-path) to Laravels `public` folder.

## Environment detection

Now what you need is, to differentiate between your local and the fortrabbit version of your Laravel App. The App should automagically be aware where it is currently running. 

We recommend to add an extra new environment variable for this purpose. You do this in the fortrabbit Dashboard › Your App › Settings › Env Vars › new environment variable.

For example, if you use fortrabbit as your production environment, set the name to `LARAVEL_ENV` with the value `prod`.

Now modify `bootstrap/start.php`:

```php
// ..
$env = $app->detectEnvironment(function () {
    return isset($_SERVER['LARAVEL_ENV'])
        ? $_SERVER['LARAVEL_ENV']
        : 'prod'; // or whatever fallback you prefer
});
// ..
```

If you use a different App on fortrabbit as [multi staging environment](multi-staging), set the same variable with the value `staging` (or whatever you prefer).

### Setting up the MySQL database

Edit your `app/config/database.php` and setup the `host`, `database`, `username` and `password` with your App's MySQL credentials. It should look something like this:

```php
return [
    'fetch'       => PDO::FETCH_CLASS,
    'default'     => 'mysql',
    'connections' => [
        'mysql'  => [
            'driver'    => 'mysql',
            'host'      => 'my-app.mysql.eu1.frbit.com',
            'database'  => 'my-app',
            'username'  => 'my-app',
            'password'  => 'YOUR-MYSQL-PASSWORD',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
        // ...
    ],
    // ...
];
```


## Advanced usage

Don't stop the body rock! Go ahead read the advanced topics and become a true master.


### Using the artisan queue

If you want to use the Artisan queue you have to enable the [Worker Add-On](workers). Once you did, you can use the `scheduler` CLI tool to setup artisan's queue to run persistently in the background.

#### Create a scheduler configuration file

Setup a new configuration file, for example in `~/htdocs/scheduler.yml`:

```yml
ArtisanQueueListener:
    type:   worker
    script: htdocs/artisan
    args:
        - "queue:listen"
```

#### Install the worker task

Login to your Worker Node via SSH, then start the Scheduler:

```bash
scheduler setup ~/htdocs/scheduler.yml
```

#### Code upgrades

Whenever you upgrade your code, make sure to restart the worker:

```bash
scheduler restart ArtisanQueueListener
```

* [Laravel and queues](http://laravel.com/docs/4.2/queues).


### Laravel 4.0 and below

With Laravel 4.1 a possible security vulnerability for certain hosting setups was closed leading to a different handling of environment detection.

Formerly, the detection was either done by the system hostname `gethostname()` or by the server name `$_SERVER['SERVER_NAME']`. Support for the latter has been dropped since 4.1.

This new environment detection does not work as you might expect and we strongly recommend to write your own detection handler based on environment variables. 
