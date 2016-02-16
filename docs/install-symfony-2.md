---

template:   article
title:      Install Symfony 2
naviTitle:  Symfony 2
lead:       Symfony has been around for some while — but it doesn't look old. Learn how to install and tune Symfony 2 on fortrabbit.
tags:
    - php
    - install
    - symfony

---


## Install

We assume you've already created a New App with fortrabbit. You also need a local Symfony installation. You can either use an existing one or initialize a new one. We further assume you want to use the environment `prod`.

For a new installation execute the following locally:

```bash
$ cd ~/Projects
$ composer create-project symfony/framework-standard-edition MyApp "2.7.*"
```

Now change into the just created, local App directory (here: `MyApp`), make sure it is initialized as a Git repo, everything is added and add your App's Git remote:

```bash
$ cd ~/Projects/MyApp
$ git init .
$ git add -A
$ git commit -m 'Initial'
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git
```

**Note**: Replace `your-app` with the name of your fortrabbit App.

Now setup the env in the [environment variable](/env-vars) in the Dashboard:

```
SYMFONY_ENV=prod
```

Once that is done, you can just push your local repo to the App remote:

```bash
$ git push -u fortrabbit master
```

This first push can take a bit, since all the Composer packages need to be installed. While that is happening, change back to the Dashboard and [set the document root](/domains#toc-set-a-custom-root-path) of your App's domains to `web`.

When the push is done you can visit your App URL in the browser and see the Symfony welcome screen! Any subsequent push will be much faster and you can leave you the `-u fortrabbit master`.

## Tuning

The above will give you an up and runnign App. However, to make the most of Symfony on fortabbit, it needs some tuning.

### Use app_dev.php

If you want to use the development, you must modify `web/app_dev.php`. A simple example would be to replace the block, responsing with a 403 like so:

```
if (isset($_SERVER['APP_NAME']) && $_SERVER['APP_NAME'] === 'my-app-name') {
    // allow
} elseif (isset($_SERVER['HTTP_CLIENT_IP'])
    || isset($_SERVER['HTTP_X_FORWARDED_FOR'])
    || !(in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', 'fe80::1', '::1')) || php_sapi_name() === 'cli-server')
) {
    header('HTTP/1.0 403 Forbidden');
    exit('You are not allowed to access this file. Check '.basename(__FILE__).' for more information.');
}
```

This way you can easily decide per App whether you want to allow the dev mode or not.

### MySQL

You can use the [App secrets](secrets) to attain your database credentials. Create the file `app/config/parameters_prod.php` with the following contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

$container->setParameter('database_driver', 'pdo_mysql');
$container->setParameter('database_host', $secrets['MYSQL']['HOST']);
$container->setParameter('database_name', $secrets['MYSQL']['DATABASE']);
$container->setParameter('database_user', $secrets['MYSQL']['USER']);
$container->setParameter('database_password', $secrets['MYSQL']['PASSWORD']);
```

Now make sure you have the above config included, for example in app/config_prod.yml:

```yaml
imports:
    - { resource: config.yml }
    - { resource: parameters_prod.php }
```

### Logging

Per default Symfony logs to a file. Modify the `app/config/config_prod.yml` file to redirect it to PHP's `error_log()`:

``` yaml
monolog:
    # ..
    handlers:
        # ..
        nested:
            type:  error_log
            # ..
```

You can safely remove `path` from the `nested` block as well. You can now use the regular [log access](logging) to view the stream.

### Migrate & other database commands

Create a new file `app/config/tunnel-db.php` and add your App's MySQL credentials:

```php
return [
    'driver'            => 'pdo_mysql',
    'database_host'     => '127.0.0.1',
    'database_port'     => '13306',
    'database_name'     => 'your-app',
    'database_user'     => 'your-app',
    'database_password' => "Your Database Password"
];
```

**Note**: Make sure not to include that file in your Git repo by adding it to `.gitgnore`!

Now open up a [tunnel](/mysql#toc-shell-tunnel-mysql) and run in another terminal window your migration commands using the `--db-configuration` parameter, eg:

```bash
$ php app/console doctrine:migrations:generate --db-configuration=app/config/tunnel-db.php
```

### Persistent storage

If you require a persistent storage, for uploaded media for example, you can use a cloud storage. We recommend Amazon's S3 and have written up a [guide to get your started](http://blog.fortrabbit.com/new-app-cloud-storage-s3). Once you've decided for a cloud storage, you can choose a file system abstraction:

* [Gaufrette Symfony bundle](https://github.com/KnpLabs/KnpGaufretteBundle)
* [Flysystem Symfony bundle](https://github.com/1up-lab/OneupFlysystemBundle)

Both are well documented. In essence, you should configure your filesystem abstraction so that you use the cloud storage adapter in your prod environment (on fortrabbit) and locally, well, a local adapter.

### Memcache sessions

If you are using a PHP production plan, then your App is distributed over multiple nodes. This means you need a network capable caching: [Memcache](memcache).

Open `app/config/parameters_prod.php` again and add:

```php
$container->setParameter('session_memcache_expire', getenv('SESSION_MEMCACHE_EXPIRE') ?: 86400);
$container->setParameter('session_memcache_prefix', getenv('SESSION_MEMCACHE_PREFIX') ?: 'ez_');
$container->setParameter('session_memcache_host_1', $secrets['MEMCACHE']['HOST1']);
$container->setParameter('session_memcache_port_1', $secrets['MEMCACHE']['PORT1']);
if (isset($secrets['MEMCACHE']['HOST2'])) {
    $container->setParameter('session_memcache_host_2', $secrets['MEMCACHE']['HOST2']);
    $container->setParameter('session_memcache_port_2', $secrets['MEMCACHE']['PORT2']);
}
```

Now open up `app/config/config_prod.yml` and make use of the parameters:

```
framework:
    session:
        handler_id: session.handler.memcached

services:
    session.memcached:
        class: Memcached
        arguments:
            persistent_id: %session_memcache_prefix%
        calls:
            - [ addServer, [ %session_memcache_host_1%, %session_memcache_port_1% ]]
            # if you are using a Memcache Production plan (with two nodes):
            #- [ addServer, [ %session_memcache_host_2%, %session_memcache_port_2% ]]

    session.handler.memcached:
        class:     Symfony\Component\HttpFoundation\Session\Storage\Handler\MemcachedSessionHandler
        arguments: [@session.memcached, { prefix: %session_memcache_prefix%, expiretime: %session_memcache_expire% }]
```

### Sending mail

You can not use [sendmail](quirks#toc-mailing) on fortrabbit but you can use the `SwiftmailerBundle` and configure it in your `app/config/config.yml` file. Make sure you set the [charset](encoding) to UTF-8:

```php
Swift_Preferences::getInstance()->setCharset('UTF-8');
```
