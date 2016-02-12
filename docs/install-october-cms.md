---

template:   article
title:      Install October CMS on fortrabbit
naviTitle:  October CMS
lead:       TODO
dontList:   true

seeAlsoLinks:
    - app
    - install-laravel-5

keywords:
    - cms
    - install
    - laravel

tags:
    - advanced
    - php
    - install

---


## Install

We assume you've already created a New App with fortrabbit which has the [MySQL](mysql), [Memcache](memcache) components enabled - optionally also the [Worker](worker) component. You also need a local [October](https://octobercms.com/docs/console/commands#console-install) installation. You can either use an existing one or start from scratch.

### Setup a new local installation

If you have a running local installation you can skip this step. Otherwise continue by executing locally:

```bash
$ cd ~/Projects
$ composer create-project october/october --prefer-dist MyApp dev-master
```

Create a new file `config/dev/database.php` with the following contents:

```php
return [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [
            'driver'    => 'mysql',
            'host'      => 'localhost',
            'port'      => '',
            'database'  => 'your-local-database-name',
            'username'  => 'your-local-database-user',
            'password'  => 'your-local-database-password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
    ],
];
```

To create the local database contents finish with:

```bash
$ php artisan october:up --env=dev
```

### Configure database settings

Create a the database configuratin file in `config/prod/database.php` with the following contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'default' => 'mysql',
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
        ],
    ],
];
```

Now create a database tunnel file, which will be needed later in this guide, under `config/tunnel/database.php` with the following contents:

```php
return [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [
            'driver'    => 'mysql',
            'host'      => '127.0.0.1',
            'port'      => 13306,
            'database'  => 'your-app-name',
            'username'  => 'your-app-name',
            'password'  => 'your-app-database-password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
    ],
];
```

**Note**: You can get your App's database credentials using the `secrets` command, eg `ssh git@deploy.eu2.frbit.com secrets your-app-name`

**Note**: This file will be later on placed in the `.gitignore` file so it will not be part of your Git history.

### Configure app settings

Create a new file `config/prod/app.php` with the follwing contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'key' => $secrets['CUSTOM']['APP_KEY'],
    'log' => 'errorlog'
];
```

### Configure cache & session settings

If you plan on running your App in a production PHP plan then you need a shared accessible cache, which Memcache is, since those plans run your App on multiple nodes. If you are just tinkering, then you can skip this topic for now.

First make sure the sessions are using your Memcache component by creating the file `config/prod/session.php` with the following content:

```php
return [
    'driver' => 'memcached'
];
```

Now you need to configure the `memcached` driver, which is the used for session and caching alike, by creating `config/prod/cache.php` containing:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

$servers = [
    ['host' => $secrets['MEMCACHE']['HOST1'], 'port' => $secrets['MEMCACHE']['PORT1'], 'weight' => 100]
];
if (isset($servers['MEMCACHE']['HOST2'])) {
    $servers []= ['host' => $secrets['MEMCACHE']['HOST2'], 'port' => $secrets['MEMCACHE']['PORT2'], 'weight' => 100];
}

return [
    'default' => 'memcached',
    'stores' => [
        'memcached' => [
            'driver'  => 'memcached',
            'servers' => $servers
        ]
    ]
];
```

### Configure filesystem settings

Since fortrabbit does not support a persistent storage you want to use S3 for that. If you don't have an AWS account or no S3 bucket handy just follow [our quick guide](http://blog.fortrabbit.com/new-app-cloud-storage-s3) and you'll have a bucket ready within a few minutes.

October comes already with Flysystem support. Flysystem abstracts the filesystem layer nicely and llows you to simply switch out the concrete adapter. Continue by installing the AWS S3 adapter:

```bash
$ composer require league/flysystem-aws-s3-v2
```

When that is done create the file `config/prod/filesystems.php` to configure an "S3 disk":

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'default' => 's3',
    'cloud' => 's3',
    'disks' => [
        's3' => [
            'driver' => 's3',
            'key'    => $secrets['CUSTOM']['AWS_S3_KEY'],
            'secret' => $secrets['CUSTOM']['AWS_S3_SECRET'],
            'region' => getenv('AWS_S3_REGION'),
            'bucket' => getenv('AWS_S3_BUCKET'),
        ]
    ]
];
```

To use the just configured disk in October you need to create `config/prod/cms.php` containing:

```php
return [
    'storage' => [
        'uploads' => [
            'disk'   => 's3',
            'folder' => 'uploads',
            'path'   => getenv('AWS_S3_URL'). '/uploads',
        ],
        'media' => [
            'disk'   => 's3',
            'folder' => 'media',
            'path'   => getenv('AWS_S3_URL'). '/media',
        ]
    ]
];
```

### Setup database

Now that all config files are created it's time to setup your App's database. Start by [opening a tunnel](mysql#toc-shell-tunnel-mysql) and then executing:

```bash
$ php artisan october:up --env=tunnel
```

### Dashboard settings

Go to the fortrabbit [Dashboard](dashboard), navigate to your App > ENV vars and create:

```
APP_ENV=prod
AWS_S3_REGION="YourAWSS3Region, eg eu-west-1"
AWS_S3_BUCKET="YourAWSS3Bucket, eg my-app-bucket"
AWS_S3_URL="YourAWSS3URL, eg https://s3-eu-west-1.amazonaws.com/my-app-bucket"
```

Now switch to App > App secrets and add:

```osterei32
APP_KEY=32CharLongRandomString
AWS_S3_KEY=YourAWSS3Key
AWS_S3_SECRET=YourAWSS3Secret
```

### Finish up

If your local app directory is not already Git controlled it's time to make it so:

```bash
$ git init .
```

Once that's done, replace the contents of the `.gitignore` file by the following:

```
/bootstrap/compiled.php
/vendor
composer.phar
.DS_Store
.idea
.env
.env.*.php
.env.php
php_errors.log
nginx-error.log
nginx-access.log
nginx-ssl.access.log
nginx-ssl.error.log
php-errors.log
sftp-config.json
selenium.php

# for netbeans
nbproject

/config/tunnel
```

(`composer.lock` must be removed and `/config/tunnel` must be added)

Finally add everything to Git, make an initial commit, add your App's remote and push:

```bash
$ git add -A
$ git commit -m 'Initial'
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git
$ git push -u fortrabbit master
```

Done

Tuning
------

### Install plugin

You need to first install the plugin locally, then refresh it using the tunnel connection (see above). The refresh makes sure the database is setup on fortrabbit as well:

```bash
php artisan --env=dev plugin:install Vendor.Name
php artisan --env=tunnel plugin:refresh Vendor.Name
```

Now add all the new plugin files to Git, commit and push:

```bash
$ git add -A
$ git commit -m 'Plugin installed'
$ git push
```

Once that's done login to your October backend and configure your plugin as usual..

### Scheduled jobs

If you plan on using scheduled jobs then you can configure a new Cron Job in the fortrabbit Dashboard > your App > Worker Jobs > Add a new Cron Job and enter:

* **Name:** `scheduler`
* **Command:** `artisan schedule:run`
* **Interval:** `every minute`

