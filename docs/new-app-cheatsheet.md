---

template:      article
title:         New App cheat sheet
naviTitle:     New App cheat sheet
noToc:         1
dontList:      1

seeAlsoLinks:
    - the-platform
    - app
    - old-and-new-apps

---


<div class="row wrap gutterbig">
    <div class="half">
        <h3>Git deployment</h3>
<pre><code># Clone your App
$ cd ~/Projects
$ git clone git@deploy.eu2.frbit.com:app-name.git

# Adding your App to existing local repo
# the name "fortrabbit" is abitrary
$ cd ~/Projects/MyProject
$ git remote add fortrabbit git@deploy.eu2.frbit.com:app-name.git

# Pushing code to your App
# 1st push
$ git push -u origin master
# 2nd to Nth push
$ git push</code></pre>
    </div>
    <div class="half">
        <h3>Composer</h3>
<pre><code class="nohighlight">As long as a `composer.json` an `composer.lock` file is present:
`composer install` will be executed on each Git push.</code></pre>
    </div>
    <div class="half">
        <h3>Deployment file example</h3>
<pre><code>version: 2

# called before composer
pre:
    
    # relative to ~/htdocs
    path: my-script.php

    # optional parameters
    args:
        - foo
        - bar
        - baz

# optional composer settings
composer:
    
    # Per default dist is prefered
    prefer-source: false
    # Resolves to the --no-plugins parameter
    no-plugins: false
    # Resolves to the --no-scripts parameter
    no-scripts: false

# called after composer
post:

    # relative to ~/htdocs
    path: sub/folder/my-script.php
    # optional parameters
    args:
        - foo

# list of sustained folders in ~/htdocs. If not given, then it defaults to the "vendor" folder.
sustained:
    - vendor</code></pre>
    </div>
    <div class="half">
        <h3>Resetting the App Git repo</h3>
<pre><code># removes existing remote Git repository
# removes vendor folder (Composer)
# will not deploy (live App stays as it is)
$ ssh git@deploy.eu2.frbit.com reset your-app-name
</code></pre>
    </div>
    <div class="half">
        <h3>Using private repositories</h3>
<pre><code># Create a keypair for your App::
$ ssh git@deploy.eu2.frbit.com keygen your-app-name
# Then copy the provided public key to Github, Bitbucket, ..
</code></pre>
    </div>
    <div class="half">
        <h3>Accessing logs</h3>
<pre><code>
# Per default all sources are tailed together:
$ ssh log@log.us0.frbit.com tail app-name

# Only Apache access log, all incoming requests with response status, time-stamp, additional headers and the first line of the request:
$ ssh log@log.us0.frbit.com tail app-name source:apache_access

# Only Apache error log, which can be very helpful to debug `.htaccess` files or the like:
$ ssh log@log.us0.frbit.com tail app-name source:apache_error

# Only PHP error logs:
$ ssh log@log.us0.frbit.com tail app-name source:web_php_error

# Only web standard errors, the output of everything written to `STDERR`:
$ ssh log@log.us0.frbit.com tail app-name source:web_stderr

# On windows with putty:
$ plink -i C:\Users\YourUser\.ssh\private.ppk log@log.eu2.frbit.com tail app-name</code></pre>
    </div>
    <div class="half">
    <h3>MySQL database credentials </h3>
<pre><code>SSH Hostname:   `tunnel.eu2.frbit.com`
SSH User:       `tunnel`
MySQL hostname: `your-app-name.mysql.eu2.frbit.com`
MySQL user:     `your-app-name`
MySQL database: `your-app-name`
MySQL password: `your-mysql-database-password`
</code></pre>
    </div>
    <div class="half">
    <h3>Accessing MySQL</h3>
<pre><code># Terminal window 1
$ ssh -N -L 13306:your-app-name.mysql.eu2.frbit.com:3306 tunnel@tunnel.eu2.frbit.com

# Terminal window 2
$ mysql -uyour-app-name -h127.0.0.1 -P13306 -p</code></pre>
    </div>
    <div class="half">
    <h3>MySQL export and import</h3>
<pre><code># Terminal window 1: Create the tunnel
$ ssh -N -L 13306:your-app-name.mysql.eu2.frbit.com:3306 tunnel@tunnel.eu2.frbit.com

# Terminal window 2: Export a dump
$ mysqldump --opt -uyour-app-name -h127.0.0.1 -P13306 -p your-app-name > your-app-name.sql

# Terminal window 3: Import a dump
$ mysql -uyour-app-name -h127.0.0.1 -P13306 -p your-app-name < your-app-name.sql</code></pre>
    </div>
</div>


