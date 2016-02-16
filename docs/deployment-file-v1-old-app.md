---

template:      article
title:         Advanced Git deployment workflows with fortrabbit.yml — version 1 for Old Apps
naviTitle:     Deployment file v1
lead:          Enhance your deployment process with the fortrabbit.yml deployment file.
dontList:      true
oldApp:        true
linkOther:     /deployment-file-v2

seeAlsoLinks:
    - deployment
    - composer
    - deployment-file-v2

tags:
   - advanced
   - git

---


This article is about the deployment file version 1, which is used by [Old Apps](old-apps). [Version 2](deployment-file-v2) is used by the New Apps.

[Git push to deploy](git) is nice and easy to get your code up and running quickly. But deployment is not deployment — there is a huge variety of use cases and preferred workflows. The `fortrabbit.yml` is an optional helper to support your habits. With the deployment file version 1 you can change the synchronization mode, add post- or pre-deploy scripts and control how Composer behaves.

## Usage

Create a file named `fortrabbit.yml` in the App's root folder of your project and include it into Git. After each push, the friendly fortrabbit bots will look for the file and parse it if existent.

### Versioning

```yml
version: 1
# configurations follow here
```

The deployment file version configuration is mandatory.



### Composer integration

Besides the optional Git [commit message](composer#toc-trigger-composer-as-a-hook) trigger, you can control when and how Composer is called using the deployment file. First you need to decide on the `method`, which can either be `install` or `update`. Then you can decide on the `mode`, which can be either `trigger` or `always`. Our recommended setup is to set the method to `install` and the mode to `always` or `trigger`. This way you can be sure that your `composer.lock` is always considered.

#### Run composer install on each push

```yml
version: 1
composer:

    # composer should use update
    method: install

    # make sure composer runs after each push
    mode: always
```

#### Run composer install when triggered

```yml
version: 1
composer:

    # still control composer via [trigger:composer]
    mode: trigger

    # make composer use install, not update
    method: install
```

#### Run composer with --prefer-source

In some rare cases, it makes sense to utilize package sources rather than package dist. Here is how you enable the `--prefer-source` flag:

```yml
version: 1
composer:

    # set the --prefer-source parameter on each install/update
    prefer-source: 1
```


### Pre- and post-deploy

For more complex deployment scenarios, you need to run code before or after the actual code synchronization takes place. The deployment file allows you to define pre- and post-deployment scripts as well as HTTP URLs which can be called before or after deployment. You might want to run code modifying your runtime data in the webspace after Git pushes. Or you just want to get an e-mail or Slack alert when someone pushes new code. Or something only you can imagine. All of this and more can be done with a Pre- or Post Deploy Hook.

TIP: You also might want to have a look at the [scripts](http://getcomposer.org/doc/articles/scripts.md) directive from composer itself.


#### Using a script

```yml
post-deploy:

    # path to the script in ~/htdocs
    script: my-script.php

    # optional list of arguments
    args: ['--env=testing']
```

Make sure the script `my-script.php` exists in the `~/htdocs` folder.

#### Using an URL

```yml
post-deploy:

    # Your custom callback URL
    url: "http://yourdomain.tld/some/path?and=args"
```

The Post Deploy Hook fires an HTTP POST request to the URL with two JSON parameters: `before` and `after`. They represent the `HEAD` commit of the Git repository before the push and after the push. Each of them contains a JSON string like the following:

```
{
    "commit" : "40cede3910db6ba0140993ae0d33376ff5df7483",
    "committer" : "author@example.com",
    "timestamp" : "1347552108",
    "changes" : {
        "my/file.html" : "M",
        "other/file.php" : "A"
    },
    "message" : "I did this and that",
    "author" : "author@example.com"
}
```

The attributes are:

* **author**: Email of the very author of the commit;
* **commit**: The Git commit hash;
* **committer**: Email of the author who has created this commit;
* **timestamp**: Unix timestamp of the date of the commit (when it was created from the author);
* **message**: Commit message;
* **changes**: Associative array containing all altered files of the commit.

You can send a success or failure message back, which will be shown in the Git push output. Therefore you have to set the _Content-Type_ to _application/json_ or _text/json_ an format the response JSON as following:

```
{
    "status": "ok",
    "message": "Some message"
}
```

The "status" has either to be `ok` or `fail`. The "message" can be arbitrary.

#### Example post deploy URL callback script

```php
// set header for json response
header('Content-type: text/json');

// protect your script with and check against the HTTP_X_FRBIT_TOKEN
define('MY_SEC_TOKEN', 'the-sec-token-of-my-app');
if (
    !isset($_SERVER['HTTP_X_FRBIT_TOKEN'])
    || $_SERVER['HTTP_X_FRBIT_TOKEN'] !== MY_SEC_TOKEN
) {
    print json_encode([
        'status'  => 'fail',
        'message' => 'No Joy!'
    ]);
    exit;
}


// decode before and after commits
$before = json_decode($_POST['before']);
$after  = json_decode($_POST['after']);


// do something
if (preg_match('/whatever/', $after->message)) {
    # ...
}


// print message
print json_encode([
    'status'  => 'ok',
    'message' => 'Alrighty'
]);
```

#### Custom parameters for your deploy script

We suggest to use the commit message as well:

```bash
# Create the commit locally
git commit -am 'My commit ### param1=ping&amp;params2=foo'
git push
```

In your script:

```php
// set header for json response
header('Content-type: text/json');

// decode before and after commits
$before = json_decode($_POST['before']);
$after  = json_decode($_POST['after']);

// parse custom parameters
$params = [];
if (preg_match('/###\s*(.+)$/', $after->message, $match)) {
    parse_str(trim($match[1]), $params);
}

// do something, write a mail
if (isset($params['param1']) && $params['param1'] === 'ping') {
    print json_encode([
        'status'  => 'ok',
        'message' => 'Pong'
    ]);
} else {
    print json_encode([
        'status'  => 'fail',
        'message' => 'Arrr!'
    ]);
}
```

#### Call multiple URLs

Just use the post-deploy-hook script instead of the URL:

```yml
post-deploy:
    script: my-url-callbacks.php
```

And then in `my-url-callbacks.php`:

```php
file_get_contents('http://my-other-domain1.tld/webcall.php');
file_get_contents('http://my-other-domain2.tld/webcall.php');
file_get_contents('http://my-other-domain3.tld/webcall.php');
```



#### Run multiple scripts after code deploy

```
version: 1

# same goes for pre-deploy
post-deploy:

    # path to the script in ~/htdocs
    script: multiple.php
```

Create a directory `post-deploy` in `htdocs` and insert here all the scripts you need.


##### multi.php example

```php
$dir = __DIR__. '/post-deploy';
foreach (glob("$dir/*.php") as $script) {
    `php $script`;
}
```

#### Call an URL after code deploy

```yml
version: 1

# same goes for pre-deploy
post-deploy:

    # Your custom callback URL
    url: "http://yourdomain.tld/some/path?and=args"
```

##### my-script.php callback script with return message

```php
header('Content-type: text/json');

if ($_SERVER['HTTP_X_FRBIT_TOKEN'] === '123123') {
    // do stuff + output
    echo json_encode(['status' => 'ok', 'message' => 'This is the message']);
} else {
    echo json_encode(['status' => 'fail', 'message' => 'Not authorized']);
}
```



### Synchronization control

With the deployment file, you can do a **fullsync** deployment, in which removed files will be deleted instead of the standard **overwrite-but-not-delete**. This implies the need of synchronization exclusions. For example, if you have an upload/ folder containing files uploaded at runtime, you don't want to delete it.

#### Update changes but don‘t delete anything

```yml
version: 1
strategy: nodelete
```

That's the default behaviour and actually doesn't need be to specified. Files that have changed will be replaced, but files you delete in Git will not be deleted on the webspace.

#### Mirror your whole Git repo

```yml
version: 1
strategy: fullsync
```

The content of the Git repo is going to be the content of the webspace afterwards.


#### Mirror your whole Git repo but exclude files

```yml
version: 1
strategy: fullsync
excludes:
    - app/storage/
    - uploads/
    - vendor/
```

Use the fullsync strategy, but exclude the folders app/storage/, uploads/ and vendor/ (aka: keep them).

**HINT**: you can use rsync via SSH to simulate the full sync deployment before running to check what would be removed. For the above example:

```bash
# change to your local project folder, containing the .git/ folder
cd my-app/

# Simulate the sync
rsync -e ssh -rlcv --dry-run --exclude=.git/ --exclude=app/storage/ --exclude=uploads/ \
    --exclude=vendor/ --delete-after ./ u-my-app@ssh1.eu1.frbit.com:~/htdocs/
```


### Multi staging settings

To allow using a single git repo with [multiple remotes](multi-staging) you can use an App name based deployment file name.

Assuming you are using two Apps, `my-app-prod` and `my-app-stage`, you can setup two deployment files with named: `fortrabbit.my-app-prod.yml` and `fortrabbit.my-app-stage.yml`. This way, you can have both deployment files in a single repo but which one is to be used is determined by the App you deploy to.

## Full Schema

<script src="https://gist.github.com/frank-laemmer/ab6a667ed79e73d15497.js"></script>
