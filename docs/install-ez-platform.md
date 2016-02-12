---

template:      article
title:         Install eZ Platform on fortrabbit
naviTitle:     eZ Platform
lead:          TODO
dontList:      true

seeAlsoLinks:
    - app

keywords:
    - ez
    - ezplatform
    - platform
    - ezpublish
    - publish
    - symfony
    - cms
    - install

tags:
    - advanced
    - php
    - install

---

Install
-------

We assume you've already created a New App with fortrabbit which has the [MySQL](mysql) and [Memcache](memcache) components enabled. You also need a local [EZ Platform](https://github.com/ezsystems/ezplatform/blob/master/INSTALL.md) installation. If you don't have one, you can setup a new one locally:

```bash
$ cd ~/Projects
$ composer create-project --prefer-dist --no-dev ezsystems/ezplatform MyApp "~1.1"
```

**Note**: Use `dev` as environment when asked. `prod` should be the environment for running on fortrabbit.

### Prepare storage

Since fortrabbit does not support a persistent storage you want to use S3 for that. If you don't have an AWS account or no S3 bucket handy just follow [our quick guide](http://blog.fortrabbit.com/new-app-cloud-storage-s3) and you'll have a bucket ready within a few minutes.

To proceed, make sure you have your bucket name, your key and your secret at your fingertips.

eZ Platform uses [Flysystem](https://github.com/thephpleague/flysystem) to handle the storage, so you need to install the [AWS PHP SDK](https://github.com/aws/aws-sdk-php) to use the S3 storage adapter:

```
$ composer require aws/aws-sdk-php
```

### Parameter config

Since you don't want any credentials in Git you need to create a PHP parameter file, which reads all credentials either from your [App's secrets](secrets) or [environment variables](env-vars). Create the `app/config/parameters_prod.php` with the following contents:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

// Database credentials
$container->setParameter('database_driver', 'pdo_mysql');
$container->setParameter('database_host', $secrets['MYSQL']['HOST']);
$container->setParameter('database_name', $secrets['MYSQL']['DATABASE']);
$container->setParameter('database_user', $secrets['MYSQL']['USER']);
$container->setParameter('database_password', $secrets['MYSQL']['PASSWORD']);

// Memcache session
$container->setParameter('session_memcache_expire', getenv('SESSION_MEMCACHE_EXPIRE') ?: 86400);
$container->setParameter('session_memcache_prefix', getenv('SESSION_MEMCACHE_PREFIX') ?: 'ez_');
$container->setParameter('session_memcache_host_1', $secrets['MEMCACHE']['HOST1']);
$container->setParameter('session_memcache_port_1', $secrets['MEMCACHE']['PORT1']);
if (isset($secrets['MEMCACHE']['HOST2'])) {
    $container->setParameter('session_memcache_host_2', $secrets['MEMCACHE']['HOST2']);
    $container->setParameter('session_memcache_port_2', $secrets['MEMCACHE']['PORT2']);
}

// AWS S3 storage credentials
$container->setParameter('aws_s3_key', $secrets['CUSTOM']['AWS_S3_KEY']);
$container->setParameter('aws_s3_secret', $secrets['CUSTOM']['AWS_S3_SECRET']);
$container->setParameter('aws_s3_region', getenv('AWS_S3_REGION'));
$container->setParameter('aws_s3_bucket', getenv('AWS_S3_BUCKET'));
$container->setParameter('aws_s3_prefix', getenv('AWS_S3_PREFIX'));

// Logging to error log
$container->setParameter('log_type', 'error_log');
```

**Note**: `app/config/parameters_prod.php` can be safely added to Git since it does not contain any actual credentials.

### General config

The general config must now include the previously created `parameters_prod.php` and further set up [Memcache](memcache) as the default session cache. Open up `app/config/config_prod.yml` and replace it with the following content:

```yaml
imports:
    - { resource: config.yml }
    - { resource: parameters_prod.php }
    - { resource: ezplatform_prod.yml }

monolog:
    handlers:
        main:
            type:         fingers_crossed
            action_level: critical
            handler:      nested
        nested:
            type:  %log_type%
            path:  %log_path%
            level: debug
        console:
            type:  console

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

### EZ platform config

Now it's time to set S3 via flysystem as the default I/O system for eZ. Replace `app/config/ezplatform_prod.yml` with:

```yaml
imports:
    - { resource: ezplatform.yml }

ezpublish:
    system:
        default:
            io:
                metadata_handler: default
                binarydata_handler: default

ez_io:
    metadata_handlers:
        default:
            flysystem:
                adapter: default
    binarydata_handlers:
        default:
            flysystem:
                adapter: default

oneup_flysystem:
    adapters:
        default:
            awss3:
                client: s3_client
                bucket: %aws_s3_bucket%
                prefix: %aws_s3_prefix%

services:
    s3_client:
        class: Aws\S3\S3Client
        factory_class: Aws\S3\S3Client
        factory_method: factory
        arguments:
            - ["%aws_s3_key%", "%aws_s3_secret%"]

```

### Dashboard settings

Once all config files are created visit the [Dashboard](dashboard) and go to your App > App secrets. Add the following secrets:

```
AWS_S3_KEY=YourAWSS3Key
AWS_S3_SECRET=YourAWSS3Secret
```

Now you need to add a couple of environment variables: Go to your App > ENV vars and add:

```
AWS_S3_REGION="YourAWSS3Region, eg eu-west-1"
AWS_S3_BUCKET="YourAWSS3Bucket, eg my-app-bucket"
AWS_S3_PREFIX="YourAWSS3Prefix, eg my-app/files"
SYMFONY_ENV=prod
```

Next go to App > Domains and set the document root of your domain(s) to the `web`.

Lastly enable the `xsl` PHP extension in App > PHP.

### Initialize database

If you started from scratch and don't have a local database, yet, it's now time to create it:

```bash
$ php app/console --env=dev ezplatform:install clean
```

Now you need to create a new environment. Let's call it `tunnel`. Begin by creating the file `app/config/config_tunnel.yml` with the following content:

```yaml
imports:
    - { resource: config.yml }
    - { resource: parameters_tunnel.php }
    - { resource: ezplatform_dev.yml }

monolog:
    handlers:
        main:
            type:         fingers_crossed
            # eZ Publish sets this to critical instead of error to avoid too verbose logs in prod
            action_level: critical
            handler:      nested
        nested:
            type:  %log_type%
            path:  %log_path%
            level: debug
        console:
            type:  console
```

Next create `app/config/parameters_tunnel.php` containing:

```php
$container->setParameter('database_driver', 'pdo_mysql');
$container->setParameter('database_host', '127.0.0.1');
$container->setParameter('database_port', 13306);
$container->setParameter('database_name', 'your-app-name');
$container->setParameter('database_user', 'your-app-name');
$container->setParameter('database_password', 'your-app-db-password');
```

**Note**: You can get your App's database credentials using the `secrets` command, eg `ssh git@deploy.eu2.frbit.com secrets your-app-name`

**Note**: You can safely add `app/config/config_tunnel.yml` to Git, but make sure that `app/config/parameters_tunnel.php` is in your `.gitignore` file, because it contains your actual credentials.

Once both files are created you should [open up a tunnel](mysql#toc-shell-tunnel-mysql) and execute:

```bash
$ php app/console --env=tunnel ezplatform:install clean
```

### Finish up

You are nearly done. Since eZ uses `app.php` as the "index file", you need to make that clear to the web server by creating `web/.htaccess`:

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ app.php [QSA,L]
DirectoryIndex app.php
```

The last file to touch is the `.gitignore` file, from which you need to remove `composer.lock` and `/web/.htaccess` as well as adding the tunnel parameter file, so your credentials are not in git

```
/vendor/
.php~
/nbproject/
/.idea/
/bin/behat
/bin/phpunit
/app/check.php
/app/SymfonyRequirements.php
/app/cache/
/app/logs/
/ezpublish_legacy
/app/config/parameters.yml
/app/bootstrap.php.cache
/web/index_treemenu.php
/web/index_rest.php
/web/index_cluster.php
/web/bundles/
/web/css/
/web/js/
/web/design
/web/extension
/web/share
/web/var
#/web/.htaccess
composer.phar
#composer.lock
.buildpath
.project
.settings/
behat.yml
.php_cs.cache
app/config/parameters_tunnel.php
```

That's about it. If your local project director is not already a Git repo, then you should make it now so:

```bash
# optional:
$ git init .

# connect with your App's remote
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git

# add all to git and push
$ git add -A
$ git commit -m 'Initial'
$ git push -u fortrabbit master
```

Done.