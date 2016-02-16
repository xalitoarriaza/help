---

template:  article
title:     Quirks & constraints
lead:      Limits, restrictions, permissions there are some — aren't there always?
dontList:  true
oldApp:    true
linkOther: /quirks

seeAlsoLinks:
    - app
    - dashboard
    - app
    - external-services
    - encoding
    - http-auth

tags:
    - beginner

---


## General

Heads up so it doesn't cost you hours of debugging in the wrong direction.

### Service location

Mind that we are currently only offering service locations in Ireland (AWS EU). For customers from the US this means some data latency as all requests have to be send over the ocean. Please also consider that all prices are in Euro (€). We are based in Berlin, our time zone is: CET.

### Mailing

Sendmail — the Mail Transfer Agent — is not available on fortrabbit.

In PHP Sendmail is usually used with the `mail()` function. Back in the good 'ol days, this would simply call the shell command `sendmail` from the shell to send a mail directly from the web server to the receiving mail server.

In recent days, this is a really bad practice: your web server can send mails, but not receive any - it is not a mail server. So it is not distinguishable from any home PC part of a bot net. Long story short: it is very likely that your mail will not be received.


#### Direct SMTP

Instead of `sendmail` you can use a mail script that uses SMTP (Simple Mail Transfer Protocol) directly. But you can't do this with fortrabbit directly either.

[Transactional mail services](external-services#toc-transactional-mails) can actually do the bulk mailing for you. You can either connect to them via "SMTP relay" or by "API". Those services help you to save your Apps resources, are probably more reliable and have some nice extra features like analytics.

In the following we focus on an SMTP implementation. There are countless possibilities how to do this. Most frameworks and CMS give them to you out of the box. If you use a custom script, have a look at [Swift Mailer](http://swiftmailer.org/).

##### SMTP & Wordpress

Use a SMTP plugin like [WP SMTP](http://wordpress.org/plugins/wp-smtp/) or [MAIL SMTP](http://wordpress.org/plugins/wp-mail-smtp/) to enable SMTP support for your `wp_mail()` function:


##### SMTP &amp; Laravel

Laravel provides a API over the popular SwiftMailer library. The mail configuration file is `app/config/mail.php`, and contains options allowing you to change your SMTP host, port, and credentials, as well as set a global from address for all messages delivered by the library.

##### SMTP & Symfony

Use the `SwiftmailerBundle` and configure it in your `app/config/config.yml` file. Make sure you set the [charset](encoding) to UTF-8:

```php
Swift_Preferences::getInstance()->setCharset('UTF-8');
```


### Logs

[Logs](directory-structure) are saved in `~/log/apache/` and `~/log/php/`. You can access the logs via SSH. The logs are written time delayed (~10-60 secs).

### Storage limit

The storage is limited, please see [pricing pages](http://www.fortrabbit.com/pricing) for details. This limit cannot be adjusted. It's anyway a good idea to use a [cloud storage](articles/external-services#toc-cloud-storage) for large data sets.

## PHP

### PATH_INFO

Our runtime implements PHP via FastCGI + FPM. To utilize `PATH_INFO` you need to do a small hack-around in your `.htaccess` file:

```
RewriteCond %{REQUEST_URI} \.php/ [NC]
RewriteRule ^(.+)\.php/(.+)$    /$1.php [NC,L,QSA,E=PATH_INFO:/$2]
```

### Basic authentication

If you want to use [HTTP basic authentication](http://en.wikipedia.org/wiki/Basic_access_authentication), you need to forward the `Authorization` header via an `.htaccess` directive and parse the contents manuall.

* [Using Basic auth on fortrabbit](http-auth)

### Authorization header

If you need the `Authorization` header, for OAuth for example, you have to forward the header explicitly via an ENV variable:

```
RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### Memory limit

Currently we limit memory to max 128 MB. The default limit is 64 MB. You can set the limit in the Settings -> PHP tab of your App. If you have disabled error output, you will only see a "white screen" when the PHP runtime reaches the limit.

### pecl_http

The PHP extension `HTTP` from PECL is enabled by default. The classes `HttpRequest`, `HttpR`sponse` and `HttpMessage` are very handy replacement for curl and alike. The downside: it breaks CakePHP in some cases. You can disable the extension in your Settings -> Extensions tab of your App.

### PHP compression

Per default compression is deactivated for PHP. Some "old" PHP applications tend to set the `Content-Length` header themselves and if the web server (Apache) compresses the file afterwards, it does not touch/modify the (now too long) content length. Here is how you enable compression back again.

Create or append to an `.htaccess` file in your document root (or a subfolder, if you want this only for a subset of your PHP scripts) with the following contents:

```
<FilesMatch \.php$>
    UnsetEnv no-gzip
</FilesMatch>
```

## Git

### Overwrite, no delete

We are using a [overwrite-but-not-delete](git) deployment. This means: when you add / change a file, and deploy your commit using git push, the file will be created / modified. When you delete a file locally, remove it from your local git repository and push your changes, nothing will be deleted from your web space (however the file is deleted from the remote repository).

### Only master branch

You can use / create as many [branches](git) as you want and push them to the fortrabbit remote repository. However, only your master branch will be deployed. This is a feature, not a bug: use other branches as "transport" branches to interchange code with other developers / locations without publishing it to your web space. Once your code is ready to deploy, just merge it in the master branch and push it.

### 2 MB file limit

Git is very good with text files. Differences can be extracted and are stored compressed. However, Git doesn't work so good with binary files. Also Git does not forget. You cannot really delete a file, once it is in the Git history. Normally our motto is to encourage, but not enforce best-practice. However in this case, we think we would do more harm than good, if we'd allow unlimited large files.

If you want to manage your binary files with Git as well, we recommend to setup a dedicated repository for this and use the [SSH + Git workflow](ssh-git).

## SSH

### Resource limits

To guarantee a high quality of service for everybody the purpose of the standard [SSH account](ssh-sftp-old-app) is limited to use (framework) CLIs and do administration operations (packing, unpacking, rsyncing). If you want to run long running scripts like workers tasks please have a look at our [Worker Nodes component](workers).


####  Limitations of standard SSH

* Process runtime is limited to -300- 600 s
* Process size is limited to -300 MB- 512 MB
* Process amount is limited to 15


### Executing scripts

You can execute PHP and bash scripts like so:

```bash
# PHP scripts
php path/to/your/script.php

# BASH scripts
bash path/to/your/script.sh
```

The file suffixes `.sh` `.php` are not required. Necessary is that you call the interpreter `php` or `bash` before the path of the script.

### The .bash_profile file

If you want to persist shell environment variables, setup aliases or just trigger something on login, the place to do this is you `.bash_profile` file in your `$HOME` folder. For example, if you want to set your primary editor to `nano` (default is `vim`), just add the following line:

```
export EDITOR=nano
```

We do not support `.bashrc`. The difference between the two is [academic](http://www.joshstaiger.org/archives/2005/07/bash_profile_vs.html) for the SSH account we provide.


## MySQL

### Persistent connections vs upgrade

If you scale (up or down) your MySQL, your App's database will be migrated to a new node. Therefore the CNAME target of your Apps MySQL hostname (`your-app.mysql.eu1.frbit.com`) will be changed. The hostname's time to live (TTL) is 60 seconds, which reflects the maximum expected downtime during upgrade.

With persistent connections this can take longer (possibly up to half an hour). Thefore we recommend to disable persistent connections during upgrades and downgrades.

## Firewalling

For [security](security) reasons, we limit outgoing traffic. Any access to services provided by us directly or via Add Ons (Memcached, MySQL, ..) is allowed. Any other traffic is per default denied - but you can request (from the Dashboard) to open ports for your App.

## Outgoing IP

Apps don't have a fixed outgoing IP address. This is a side effect of "the cloud", as Apps need to be capable of moving fast between nodes, due to failover, scaling or load balancing.

As we are currently only in the AWS EU1 (Ireland) region, there is an semi-official [list of ip ranges](https://forums.aws.amazon.com/ann.jspa?annID=1701), which we are using. Depending on the use-case, it is possible to use a [HTTP](https://www.quotaguard.com/pricing#_quotaguardstatic) [proxy](http://www.vpnuk.info/dedicated-ip.html) provider, which offers a static IP.

The context for requests on this is for payment processing, have a look at an [external provider](external-services#toc-payment-processing).

