---

template:   article
title:      Install Laravel 5
naviTitle:  Laravel 5
lead:       Laravel is the most PHPopular framework. Learn how to install and tune Laravel 5 on fortrabbit.
linkOther:  /install-laravel-5-old-app
tags:
    - php
    - install
    - laravel

---

Install
-------

We assume you've already created a New App with fortrabbit. You also need a local [Laravel](http://laravel.com/docs/5.1/installation) installation. You can either use an existing one or initialize a new one. For a new one execute locally:

```bash
$ cd ~/Projects
$ composer create-project laravel/laravel --prefer-dist MyApp
```

In any case: change into your local app directory, make sure it is initialized as a Git repo, everything is added and add your App's Git remote:

```bash
$ cd ~/Projects/MyApp
$ git init .
$ git add -A
$ git commit -m 'Initial'
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git
```

**Note**: Replace `your-app` with the name of your fortrabbit App.

If you haven't already created a key for your Laravel installation, you can do it now by running locally:

```bash
$ php artisan key:generate
```

This will print out the key and write it to your local `.env` file. You can now either create another key or use that one and set is as an [environment variable](/env-vars) to your App, via the Dashboard.

Once that is done, you can just push your local repo to the App remote:

```bash
$ git push -u fortrabbit master
```

This first push can take a bit, since all the Composer packages need to be installed. While that is happening, change back to the Dashboard and [set the document root](/domains#toc-set-a-custom-root-path) of your App's domains to `public`.

When the push is done you can visit your App URL in the browser and see the Laravel welcome screen! Any subsequent push will be much faster and you can leave you the `-u fortrabbit master`.

Tuning
------

The above will give you an up and running App. However, to make the most of Laravel on fortabbit, it needs some tuning.

### Logging

Per default Laravel writes all logs to `storage/log/..`. Since you don't have direct file access, you need to configure Laravel to write to the PHP `error_log` method instead. That's easily done: open `boostrap/app.php` and add the following just before the `return $app` statement at the bottom:

```php
$app->configureMonologUsing(function($monolog) {
    // chose the error_log handler
    $handler = new \Monolog\Handler\ErrorLogHandler();
    // time will already be logged -> change default format
    $handler->setFormatter(new \Monolog\Formatter\LineFormatter('%channel%.%level_name%: %message% %context% %extra%'));
    $monolog->pushHandler($handler);
});
```

You can now use the regular [log access](logging) to view the stream.

### MySQL

You can use the [App secrets](secrets) to attain your database credentials. Modify the `config/database.php` like so:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

// ..
return [
    // ..
    'connections' => [
        // ..
        'mysql' => [
            'driver'    => 'mysql',
            'host'      => $secrets['MYSQL']['HOST'],
            'port'      => $secrets['MYSQL']['PORT'],
            'database'  => $secrets['MYSQL']['DATABASE'],
            'username'  => $secrets['MYSQL']['USER'],
            'password'  => $secrets['MYSQL']['PASSWORD'],
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
            'strict'    => false,
        ],
        // ..
    ],
    // ..
];
```


#### Setting time zone in Laravel

As Eloquent uses `PDO`, you can use the `PDO::MYSQL_ATTR_INIT_COMMAND` option. Extend your `mysql` configuration array in `app/config/database.php` or your specific environment `database.php` file:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    // ..
    'mysql' => [
        'driver'    => 'mysql',
        'host'      => $secrets['MYSQL']['HOST'],
        'database'  => $secrets['MYSQL']['DATABASE'],
        'username'  => $secrets['MYSQL']['USER'],
        'password'  => $secrets['MYSQL']['PASSWORD'],
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
        'options'   => [
            // Here is the time zone setting
            \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET time_zone = \'+02:00\''
        ]
    ],
    // ..
];
```




### Memcache

Best to use the [App secrets](app-secrets) to attain your database credentials. Modify the `memcached` connection in your `config/cache.php` like so:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

// ..
return [
    // ..
    'stores' => [
        // ..
        'memcached' => [
            'driver'  => 'memcached',
            'servers' => $secrets['MEMCACHE']['COUNT'] == 1
                ? [
                    ['host' => $secrets['MEMCACHE']['HOST1'], 'port' => $secrets['MEMCACHE']['PORT1'], 'weight' => 100],
                ]
                : [
                    ['host' => $secrets['MEMCACHE']['HOST1'], 'port' => $secrets['MEMCACHE']['PORT1'], 'weight' => 100],
                    ['host' => $secrets['MEMCACHE']['HOST2'], 'port' => $secrets['MEMCACHE']['PORT2'], 'weight' => 100],
                ],
        ],
        // ..
    ],
    // ..
];
```

In addition, set the `CACHE_DRIVER` [environment variable](env-vars) so that you can use `memcached` in your production App on fortrabbit and `apc` or `array` on your local machine, via `.env` file.

### Migrate & other database commands

Open `config/database.php` and add a new connection:

```php
// ..
'connections' => [
    // ..
    'mysql-tunnel' => [
        'driver'    => 'mysql',
        'host'      => '127.0.0.1',
        'port'      => '13306',
        'database'  => 'your-app-name',
        'username'  => 'your-app-name',
        'password'  => env('DB_PASSWORD', ''),
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
        'strict'    => false,
    ],
    // ..
],
```

Now open up a [tunnel](/mysql#toc-shell-tunnel-mysql) and run in another terminal window your migration or seed commands:

```bash
$ DB_PASSWORD="your database password" php artisan migrate --database=mysql-tunnel
$ DB_PASSWORD="your database password" php artisan db:seed --database=mysql-tunnel
```

### Persistent storage

If you require a persistent storage, for uploaded media or anything else, you can use a cloud storage. We recommend Amazon's S3 and have written up a BLOG[guide to get your started](new-app-cloud-storage-s3).

For the sake of an example, let's say you are using S3. First setup the `S3_*` [custom App secrets](/secrets#toc-adding-custom-app-secrets). Then you can open up `config/filesystems.php` and modify it akin to the following:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    // ..
    'disks' => [
        // ..
        's3' => [
            'driver' => 's3',
            'key'    => $secrets['CUSTOM']['S3_KEY'],
            'secret' => $secrets['CUSTOM']['S3_SECRET'],
            'region' => $secrets['CUSTOM']['S3_REGION'],
            'bucket' => $secrets['CUSTOM']['S3_BUCKET'],
        ],
        // ..
    ],
    // ..
];
```

If you want to use S3 remotely and a local storage locally, then replace the "default" value. For example like so:

```php
// ...
'default' => env('FS_TYPE', 'local'),
// ..
```

Now set `FS_TYPE` in your local `.env` file and the [environment variables](/env-vars) in the Dashboard.



### Sending mail

You can not use [sendmail](quirks#toc-mailing) on fortrabbit but Laravel provides a API over the popular SwiftMailer library. The mail configuration file is `app/config/mail.php`, and contains options allowing you to change your SMTP host, port, and credentials, as well as set a global form address for all messages delivered by the library.

