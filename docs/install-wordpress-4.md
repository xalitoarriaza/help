---

template:   article
title:      Install WordPress 4
naviTitle:  WordPress 4
lead:       WordPHPress is PHPowering much of the web. Learn here how to install and tune the popular blogging and CMS engine WordPress 4 on fortrabbit.
linkOther:  /install-wordpress-old-app
tags:
    - php
    - install
    - wordpress

seeAlsoLinks:
    - app

---


Install
-------

This guide has been tested with WordPress 4.3 and 4.4.

WordPress itself has not really arrived in the new PHP age in terms of using the latest or even recent technologies and paradigms. But do not fear: [Bedrock](https://roots.io/bedrock/) comes to the rescue. Bedrock is a "WordPress boilerplate with modern development tools, easier configuration, and an improved folder structure" and allows you to install WordPress easily in a modern infrastructure, such as fortrabbit.

Let's get started by installing Bedrock locally:

```bash
# create a new App based on bedrock locally
$ cd ~/Projects
$ git clone https://github.com/roots/bedrock MyApp

# add your App's remote, so you can push later on
$ cd MyApp
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git

# install all required composer packages
$ composer install
```

Now head over to the Dashboard, [set the document root](/domains#toc-set-a-custom-root-path) of your App's domains to `web` and then add the following [App secrets](secrets):

```osterei32
AUTH_KEY=LongRandomString
SECURE_AUTH_KEY=LongRandomString
LOGGED_IN_KEY=LongRandomString
NONCE_KEY=LongRandomString
AUTH_SALT=LongRandomString
SECURE_AUTH_SALT=LongRandomString
LOGGED_IN_SALT=LongRandomString
NONCE_SALT=LongRandomString
```

WordPress recommends replace each `LongRandomString` with a *different* random string. The last configuration in the Dashboard is to set the WordPress URLs and environment as [environment variables](env-vars):

```
WP_ENV=production
WP_HOME=http://your-app.eu2.frbit.net
WP_SITEURL=http://your-app.eu2.frbit.net/wp
```

Now open up the main config file in `config/application.php` and the following at the very top of the file:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);
```

Go to the `DB settings` section and replace the `getenv("DB_*")` calls with access to the MySQL secrets:

```php
define('DB_NAME', $secrets['MYSQL']['DATABASE']);
define('DB_USER', $secrets['MYSQL']['USER']);
define('DB_PASSWORD', $secrets['MYSQL']['PASSWORD']);
define('DB_HOST', $secrets['MYSQL']['HOST']);
```

Then do the same for the `Authentication Unique Keys and Salts` section:

```php
define('AUTH_KEY', $secrets['CUSTOM']['AUTH_KEY']);
define('SECURE_AUTH_KEY', $secrets['CUSTOM']['SECURE_AUTH_KEY']);
define('LOGGED_IN_KEY', $secrets['CUSTOM']['LOGGED_IN_KEY']);
define('NONCE_KEY', $secrets['CUSTOM']['NONCE_KEY']);
define('AUTH_SALT', $secrets['CUSTOM']['AUTH_SALT']);
define('SECURE_AUTH_SALT', $secrets['CUSTOM']['SECURE_AUTH_SALT']);
define('LOGGED_IN_SALT', $secrets['CUSTOM']['LOGGED_IN_SALT']);
define('NONCE_SALT', $secrets['CUSTOM']['NONCE_SALT']);
```

Before visting your site, push all the changes:

``` bash
$ git commit -am "Initial"
$ git push -u fortrabbit master
```

Finally you can visit your App in the Browser and follow the WordPress setup. Done.

Tuning
------

### Installing plugins

[Bedrock allows](https://github.com/roots/bedrock/wiki/Composer#plugins) two ways to install plugins:  Use Composer to install plugins from [WordPress Packagist](http://wpackagist.org/); Just download and unpack the plugins into `web/app/plugins`

#### With Composer

This is the recommend way, since it already provides a simple upgrade mechanism (Composer, of course). Following is an example on how to install Akismet:

```bash
$ composer require wpackagist-plugin/akismet
$ git commit -am "With Akismet"
$ git push
```

Now you should enable the Plugin via the WordPress admin. Not all plugins are, as of yet, available via Wordpress Packagist, but the number is growing. Use their [search](http://wpackagist.org/) to check if the plugin you require is available.

#### Without Composer

If the plugin is not (yet) available via WordPress Packagist, you can still use the "old way" to install plugins. Following an example to install the WooCommerce plugin:

```bash
$ cd ~/Projects/MyApp/web/app/plugins
$ wget http://downloads.wordpress.org/plugin/woocommerce.zip
$ unzip woocommerce.zip
$ rm woocommerce.zip
```

Now make sure that the WooCommerce plugin directory is excluded from the `.gitignore` (aka: un-ignored) by opening `~/Projects/MyApp/.gitignore` and adding a new line immediately after `!web/app/plugins/.gitkeep`:

```
# ...
web/app/plugins/*
!web/app/plugins/.gitkeep
!web/app/plugins/woocommerce
# ...
```

Now you can add everything to Git and push it online:

```bash
$ cd ~/Projects/MyApp
$ git add -A
$ git commit -m "With WooCommerce"
$ git push
```

Once that's done you can head over your WordPress admin and activate the plugin as usual.

### Installing themes

Bedrock recommends to install themes by unpacking them into the `web/app/themes` folder and to use Composer only in [rare cases](https://github.com/roots/bedrock/wiki/Composer#themes).

### Persistent storage

Since WordPress is a CMS living on editorial provided content you most likely need a persistent storage. That's not so hard.

First you need an Amazon Web Services (AWS) account, then you simply install two plugins and configure them. The AWS account can be easily obtained. Just follow the instructions in BLOG[our guide to S3](new-app-cloud-storage-s3) - should not take more than a couple of minutes.

Once that's done you need to install two plugins. Best do it with composer:

```bash
$ composer require wpackagist-plugin/amazon-web-services
$ composer require deliciousbrains/wp-amazon-s3-and-cloudfront
```

Now switch to the fortrabbit Dashboard and add two environment [secrets](secrets):

```
AWS_ACCESS_KEY_ID=YourAWSAccessKey
AWS_SECRET_ACCESS_KEY=YourAWSSecretKey
```

Next open your environment config, for example `config/environments/production.php`, and add:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

define('AWS_ACCESS_KEY_ID', $secrets['CUSTOM']['AWS_ACCESS_KEY_ID']);
define('AWS_SECRET_ACCESS_KEY', $secrets['CUSTOM']['AWS_ACCESS_KEY_ID']);
```

Now commit everything and push it online:

```bash
$ git commit -am "With S3"
$ git push
```

Once that's done, you can head over to the WordPress Admin, activate the two previously installed plugins ("Amazon Web Services" and "WP Offload S3") and then go to the settings of "WP Offload S3". Here you can choose one of your buckets. And, besides some optional fine-tuning, that's it. Any media you will upload hence will be persisted to S3.

### Sending mail

You can not use [sendmail](quirks#toc-mailing) on fortrabbit but you can use a SMTP plugin like [WP SMTP](http://wordpress.org/plugins/wp-smtp/) or [MAIL SMTP](http://wordpress.org/plugins/wp-mail-smtp/) to enable SMTP support for your `wp_mail()` function:
