---

template:   article
title:      Install Laravel 4
naviTitle:  Laravel 4
lead:       Laravel is the most PHPopular framework. Learn how to install and tune Laravel 4 on fortrabbit.
linkOther:  /install-laravel-4-old-app
tags:
    - php
    - install
    - laravel

---

Install
-------

We assume you've already created a New App with fortrabbit. You also need a local [Laravel](http://laravel.com/docs/4.2/installation) installation. You can either use an existing one or intialize a new one. For a new one execute locally:

```bash
$ cd ~/Projects
$ composer create-project laravel/laravel --prefer-dist MyApp 4.2
```

In any case: Change into your local app directory, make sure it is initialized as a Git repo and add your fortrabbit App remote:

```bash
$ cd ~/Projects/MyApp
$ git init .
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git
```

**Note**: Replace `your-app` with the name of your fortrabbit App.

Now open `composer.json` and modify the `post-install-cmd` under `scripts` by adding `"test -d app/storage || mkdir app/storage"` as the first item:

```json
    // ..
    "post-install-cmd": [
        "test -d app/storage || mkdir app/storage",
        "test -d app/storage/cache || mkdir app/storage/cache",
        "test -d app/storage/logs || mkdir app/storage/logs",
        "test -d app/storage/meta || mkdir app/storage/meta",
        "test -d app/storage/sessions || mkdir app/storage/sessions",
        "test -d app/storage/views || mkdir app/storage/views",
        "php artisan clear-compiled",
        "php artisan optimize"
    ],
    // ..
```

Next open `.gitignore` and remove the `composer.lock` line. Once all is done, you can add everything, create an initial commit and push your local repo to the App remote:

```bash
$ git add -A
$ git commit -m 'Initial'
$ git push -u fortrabbit master
```

This first push can take a bit, since all the Composer packages need to be installed. While that is happening, change back to the Dashboard and [set the document root](/domains#toc-set-a-custom-root-path) of your App's domains to `public`.

When the push is done you can visit your App URL in the browser and see the Laravel welcome screen! Any subsequent push will be much faster and you can leave you the `-u fortrabbit master`.

Tuning
------

The above will give you an up an running App. However, to make the most of Laravel on fortabbit, it needs some tuning.

### Environment detection

Laravel 4 supports multiple environments, which then can have custom configurations in `app/config/<env-name>/something.php`. When running your App locally, you want to use the `local` environment. When running on fortrabbit, you want a different environment, namely `production`.

For this open `boostrap/start.php` and replace the whole `detectEnvironment` block by the following:

```php
$env = $app->detectEnvironment(function() {
    if (isset($_SERVER['APP_NAME']) && $_SERVER['APP_NAME']) {
        return 'production';
    } else {
        return 'local';
    }
});
```

### Logging

Per default Laravel writes all logs to `app/storage/log/..`. Since you don't have direct file access, you need to configure Laravel to write to the PHP `error_log` method instead. That's easily done: Open `boostrap/start.php` and add the following just before the `return $app` statement at the bottom:

```php
if ($env == 'production') {
    $logHandler = new \Monolog\Handler\ErrorLogHandler();
    $logHandler->setFormatter(new \Monolog\Formatter\LineFormatter('%channel%.%level_name%: %message% %context% %extra%'));
    Log::getMonolog()->pushHandler($logHandler);
}
```

You can now use the regular [log access](logging) to view the stream.


### Detect HTTPS

If you want to use `Request::secure()` or the like, to detect whether the current connect is secure, then you need to add the following early on in in your `app/filters.php`:

```php
// ..
App::before(function($request) {
    if (strtolower($request->header('X-Forwarded-Proto') ?: '') == 'https') {
        $request->setTrustedProxies([$request->getClientIp()]);
    }
    // ..
});
// ..
```

### MySQL

Best practice would be to utilize Laravel environments (see above): Create `app/config/production/database.php` and leverage [App secrets](secrets) to attain your database credentials:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'connections' => [
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
        ]
    ]
];
```

#### Setting a custom time zone

As Eloquent uses `PDO`, you can use the `PDO::MYSQL_ATTR_INIT_COMMAND` option. Extend your `mysql` configuration from above:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'connections' => [
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

            // Here is the time zone setting
            'options'   => [
                \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET time_zone = \'+02:00\''
            ]
        ]
    ]
];
```




### Memcache

Best to use the [App secrets](app-secrets) to attain your Memcache credentials. Create a config for your production environment (see above) for the `memcached` connection in `app/config/production/cache.php`:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'driver'    => 'memcached',
    'prefix'    => 'laravel',
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
];
```

### Migrate & other database commands

Create a connection to your local environment database config by opening adding the following to `app/config/local/database.php`:

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
