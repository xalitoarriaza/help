---

template:   article
title:      Using secure App secrets
naviTitle:  App secrets
lead:       App secrets provide a secure storage for all the credentials your App needs to run.

keywords:
    - Secrets
    - Credentials
    - ENV vars
    - Environment variables

seeAlsoLinks:
   - env-vars

tags:
  - php
  - beginner

---

## Problem

Your App needs confidential credentials to connect to other services (user-names, passwords, API keys or alike). Those credentials can be on fortrabbit like MySQL database access, or externally like the Amazon S3 cloud storage API key.

You [should not store them within your code base](//blog.fortrabbit.com/how-to-keep-a-secret#not-in-git), since it's controlled by Git - so version controlled. You [should not store them unencrypted in ENV vars](//blog.fortrabbit.com/how-to-keep-a-secret#not-in-an-env-var-either-) either.

In addition: your App will run in at least two environments: locally and on fortrabbit. Any solution must address the problems resulting from multiple runtime environments.

## Solution

Use fortrabbits App secrets to store your credentials safely. App secrets are stored in a JSON file which is only accessible by your App. The location of this JSON file is stored in the environment variable `APP_SECRETS`.

## App secrets in PHP

```php
// read all secrets
$secrets = json_decode(file_get_contents($_SERVER["APP_SECRETS"]), true);

// access a specific secret
echo $secrets['MYSQL']['HOST'];
```

The secrets are ordered in a tree structure in the format `["CONTEXT" => ["KEY" => "value"]]`, for example:

```php
$secrets == [
    'MYSQL' => [
        'PASSWORD' => "Your App's MySQL password",
        'USER'     => "Your App's MySQL user name",
        'DATABASE' => "Your App's MySQL database name",
        'HOST'     => "Your App's MySQL hostname",
        'PORT'     => "Your App's MySQL port",
    ],
    'CUSTOM' => [
        'YOUR_CUSTOM_SECRET'    => "The custom content",
        'yourOtherCustomSecret' => "The custom content"
    ]
];
```

Which contexts are available depends on which add-ons you have enabled for your App.

## Adding custom App secrets

You can add or remove custom App secrets in the [Dashboard](dashboard). You'll do so in the settings of your [App](app). The contents of the App secrets cannot be viewed in the Dashboard due to the underlying encryption, which we consider a feature, not a bug.

## Accessing App secrets

Besides accessing the secrets programatically - as shown above - you can view all or a subset of them using the the command line:

```bash
# read all secrets
$ ssh git@deploy.eu2.frbit.com secrets your-app
{
    "MYSQL": {
        "PASSWORD": "AAAABBBBCCCCDDDDEEEEFFFF",
        "USER": "my-app",
        "DATABASE": "my-app",
        "HOST": "my-app.mysql.eu2.frbit.com",
        "PORT": "3306",
    },
    "CUSTOM": {
        "YOUR_CUSTOM_SECRET": "The custom content",
        "yourOtherCustomSecret": "The custom content"
    }
}

# read only MySQL secrets
$ ssh git@deploy.eu2.frbit.com secrets your-app mysql
{
    "PASSWORD": "AAAABBBBCCCCDDDDEEEEFFFF",
    "USER": "my-app",
    "DATABASE": "my-app",
    "HOST": "my-app.mysql.eu2.frbit.com",
    "PORT": "3306",
}

# read only MySQL password
$ ssh git@deploy.eu2.frbit.com secrets your-app mysql.password
AAAABBBBCCCCDDDDEEEEFFFF
```

## App secrets vs ENV vars

App secrets are closely related to ENV vars insofar that they are both available to your App at runtime. The big difference between them is that App secrets are stored highly secured and they are not automatically dumped out by debug tools - such as `phpinfo()` or your favorite debug toolbar.

To achieve a level of security approaching the App secrets with ENV vars you can encrypt them, as described in [the ENV var article](env-vars#toc-env-vars-vs-security).

## App secrets vs local environment

Since access to your App secrets should be done using `$_SERVER['APP_SECRETS']`, you can easily set this environment variable locally with a path to a local JSON file containing your local (dummy) secrets.