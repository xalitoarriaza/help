---

template:      article
title:         Install Craft CMS 2 on fortrabbit
naviTitle:     Craft CMS 2
lead:          Craft is a CMS you and your clients love. Learn how to deploy Craft using git on fortrabbit.

keywords:
    - craft
    - craftCMS
    - setup
    - install-guide

tags:
    - advanced
    - php
    - install

externalLinks:
    - https://craftcms.com/

---

Install
-------

This guide has been tested with Craft CMS 2.5.

We assume you've already created an App with fortrabbit and that you have a local [Craft](https://craftcms.com/docs/installing) installation running. 

### Requirements

Before you get started you need a local, running Craft installation. If you are starting from scratch then best use the [HappyLager Demo](https://github.com/pixelandtonic/HappyLager) and follow their install guide. If you have a running (production) installation then you need export to its data and set up a local, working "clone" with which you can proceed.

### Setup asset source

Since fortrabbit does not support a persistent storage you want to use S3 for that. If you don't have an AWS account or no S3 bucket handy just follow [our quick guide](http://blog.fortrabbit.com/new-app-cloud-storage-s3) and you'll have a bucket ready within a few minutes.

Once that's done your can create you S3 asset source:

* Go to Settings > Assets
* Click on `New asset source` to create a new `AssetSource` 
* Give it a name and set the Type to `Amazon S3` 
* Enter the S3 Access Key ID and Secret Access Key click on `Refresh`
* Select a `Bucket`
* Enter a `Subfolder` (we prefer to put the App's name here)
* Set `Cache Duration` to 1 hour (you can tune that later)
* Click `Save` in the upper right corner
* Done

Now, since you are working on a local copy which you want to deploy to your fortrabbit App you now need to move all the "local assets" to the newly created S3 asset source:

* Go to Assets
* Drag and drop all your assets to the "Amazon S3" asset source on the left side
* Done

**Note**: Rinse and repeat with all your local asset sources!

**Note:** To make use of the cloud storage suppport of Craft you need "Pro" license. This is required since fortrabbit's filesystem is not persistent and assets needs a place, too.

Now that is done you can safely remove the empty, local asset sources:

* Got to Settings > Assets
* Click the remove button for every `Local Folder` type asset source
* Done

### Migrate database

Database migration is straight forward: Export the database of the local installation and import it to your fortrabbit App. You can use a [GUI](mysql#toc-mysql-guis) or the shell. From the shell you start out by [opening up a tunnel](mysql#toc-shell-tunnel-mysql) and then use `mysqldump` to export and `mysql` to import:

```bash
# on your local machine

# export from local
$ mysqldump database-name > database.sql

# import to fortrabbit using the tunnel
$ mysql -h127.0.0.1 -P13306 -umy-app -p my-app < database.sql
```

### Configuration files

Craft's native multi-environment configuration options, which allow you to define configuration options based on the domain name. This is great, but there is a potential security flaw when using Git based deployments you should be aware of: You're hard-coding the configuration details of your production environment into your code, which means you will have sensitive information in your Git version history.

Do not fear, that's easily solved:

#### General

Login the fortrabbit Dashboard and set the following [environment variables](env-vars):

```
CRAFT_DEBUG=0
CRAFT_CACHE=memcache
CRAFT_UPDATES=0
```
Then, still in the fortrabbit Dashboard, head over to the [App secrets](secrets) and add a validation key (long random string):

```osterei32
CRAFT_KEY=LongRandomString
```

Also, while you are at it, head over to your domains and set the document root to `public`.

Now open up `craft/config/general.php` and change it like so:

```php
$validationKey = false;
if ($file = getenv('APP_SECRETS')) {
    $secrets = json_decode(file_get_contents($file), true);
    $validationKey = $secrets['CUSTOM']['CRAFT_KEY'];
}

return [
    'allowAutoUpdates'  => getenv('CRAFT_UPDATES') ?: true,
    'cacheMethod'       => getenv('CRAFT_CACHE') ?: 'file',
    'devMode'           => getenv('CRAFT_DEBUG') ?: false,
    'siteUrl'           => 'http://'. $_SERVER['HTTP_HOST'],
    'validationKey'     => $validationKey,
    'environmentVariables' => [
        'assetsBaseUrl'  => '/assets',
        'assetsBasePath' => './assets',
    ]
];
```

#### Database

Open `craft/config/db.php` and modify it like the following:

```php
// production environment (fortrabbit)
if ($file = getenv('APP_SECRETS')) {
    $secrets = json_decode(file_get_contents($file), true);
    return [
        'server'      => $secrets['MYSQL']['HOST'],
        'user'        => $secrets['MYSQL']['USER'],
        'password'    => $secrets['MYSQL']['PASSWORD'],
        'database'    => $secrets['MYSQL']['DATABASE'],
        'port'        => $secrets['MYSQL']['PORT'],
        'tablePrefix' => 'craft',
    ];
}

// development environment (local)
return [
    'server'      => '127.0.0.1',
    'user'        => 'your-local-db-user',
    'password'    => 'your-local-db-password',
    'database'    => 'your-local-db-name',
    'tablePrefix' => 'craft',
];
```

#### Cache & Session

If you are just testing out Craft the make sure you use tinkering PHP plan. With those you can set the cache variable (above) to `file`, `db` or `apc` during your tinkering, for example:

```
CRAFT_CACHE=db
```

All fortrabbit production PHP plans run on multiple nodes and require a cache which is accessible jointly - same goes for sessions, of course. This disqualifies `apc`, `db` and `file` and leaves you with `memcache` as options.

So make sure you have booked the [Memcache component](memcache) and then create the file `craft/config/memcache.php` with the following content:

```php
if ($file = getenv('APP_SECRETS')) {
    $secrets = json_decode(file_get_contents($file), true);
    if (isset($secrets['MEMCACHE'])) {
        $memcache = $secrets['MEMCACHE'];

        $handlers = [];
        $servers  = [];
        foreach (range(1, $memcache['COUNT']) as $num) {
            $handlers []= $memcache['HOST'. $num]. ':'. $memcache['PORT'. $num];
            $servers []= [
                'host'          => $memcache['HOST'. $num],
                'port'          => $memcache['PORT'. $num],
                'retryInterval' => 2,
                'status'        => true,
                'timeout'       => 2,
                'weight'        => 1,
            ];
        }

        // session config
        ini_set('session.save_handler', 'memcached');
        ini_set('session.save_path', implode(',', $handlers));
        if ($memcache['COUNT'] == 2) {
            ini_set('memcached.sess_number_of_replicas', 1);
            ini_set('memcached.sess_consistent_hash', 1);
            ini_set('memcached.sess_binary', 1);
        }

        // craft config
        return [
            'servers'      => $servers,
            'useMemcached' => true,
        ];
    }
}
```


### Deploy with Git

If your local Craft installation is not under Git version control, then you do it now:

```
$ cd ~/Projects/MyApp
$ git init .
```

Add your App's Git remote to your local repo:

```
$ git remote add fortrabbit git@deploy.eu2.frbit.com:my-app.git
```

Now open and modify the `.gitignore` file:

```bash
# Exclude misc project files
*.idea/*
*.log
*.DS_Store
*Thumbs.db
*.sass-cache/
node_modules
*config.codekit
*.sql

# Exclude uploads, runtime data and temp files, but keep the layout assets
/craft/storage/runtime/
/craft/storage/cache/
/craft/storage/logs/
/craft/storage/compiled_templates/
/public/assets/*
!/public/assets/css/*
!/public/assets/js/*
!/public/assets/fonts/*

# Exclude Composer's vendor folder
/vendor/
```

You're almost done with the basic setup. Last step is to deploy your code:

```
$ git add -A
$ git commit -m 'Init'
$ git push -u fortrabbit master
```

Done! Your Craft App is online!

Advanced Topics / Tuning
-------

### Logging & Debugging

Per default Craft writes all the logs to a file, which won't do. So we wrote a simple plugin, which writes all logs to STDERR instead. Then the logs are accessible via our standard [logging](logging).

The plugin can be found here: [github.com/ostark/craft-stderr-logger](https://github.com/ostark/craft-stderr-logger)



### Sending mail

You can not use [sendmail](quirks#toc-mailing) on fortrabbit but Craft provides multiple alternative options. Just go to Settings > Email, change the `Protocol` to `SMTP` or `Gmail` and enter the credentials of your mail service.

