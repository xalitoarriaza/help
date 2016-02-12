---

template:      article
title:         How to work with the New Apps
naviTitle:     New Apps
lead:          Learn how the New Apps differ from the Old Apps in practical use.

---

We have two generations of Apps here on fortrabbit: [Old Apps](old-apps) (2013 - mid 2016) and New Apps (from 2015 on). The New Apps are a bit different to the Old Apps and not fully backwards compatible. This article here is a look on what changes in practical day by day usage in contrast to Old Apps.




## When to migrate form Old App to New App

Should you migrate your Old Apps to New Apps today? Yes, good idea. The New Apps are nearly feature complete. Only one component is missing and you might not even need it - or can used an alternative already. The missing component is:

<!-- TODO: refactor on asset storage launch -->

### Asset storage

Old Apps have a **persistent storage**: you have access via SSH and the App can permanently write on the file system. New Apps have an **ephemeral storage**: you have no SSH access, all files will be replaced on each deployment. There is no place for runtime data. In other words: your App should not write any relevant data (user uploads) you might want to reuse later on the local file system. 

The upside is, it's a design that scales much better. The downside is, that you'll need some efforts from your side. Currently you can use an external [cloud storage provider](external-services#toc-cloud-storage). In the future you will offer an integrated service with fortrabbit.


### Communication

Don't worry to miss something. We are aware that this is a sensitive topic. We are in this together.  We will not just switch off your live Apps. We will look after each single App.

Migration will take some efforts from your side, but it will be worth it. You will benefit form less costs, better performance and a better experience. 

The "missing" Components will come soon. Then we'll also publish more detailed informations on how to migrate, a general article is already [over here](/migrating). We will also communicate informations and deadlines upfront. We will give you extensive support on this. We will not rush things, there will be enough time. We will not let you down.







## Only Git, no SSH/SFTP

The Old Apps offered [Git](git) and [SSH](ssh-sftp-old-app) deployment. With the New Apps you only deploy with Git. That means you have no longer direct file access to the actual "webspace" of your App.

## Atomic deployment

The Old Apps Git offer a synchronization deployment mode: the Git repo contents get's rsynced file by file. Per default this is non-destructive, new files and changes would replace old files, but no files will be deleted (overwrite but not delete).

The New Apps deployment is [atomic](http://blog.fortrabbit.com/new-apps-are-here). That means that a completely new release package will substitute the old one on every deploy. We have made a [behind the scenes video](deployment-architecture-video) showcasing what is happening in the background. The atomic deployment features also a true build process: your code will only be released if all scripts (composer install, optional pre- and post-deploy scripts) succeed.

Everything will feel familiar. You will most likely you'll first notice a different more detailed Git deploy log.


### Composer packages

We recommend to use a fileystem abstraction, which allows you to easily swap out local with remote storage. They provide multiple adapters to virtually any cloud storage (AWS S3, Azure, Rackspace, Dropbox, Copy.com …). The most widely used and battle proven are:

* [flysystem](https://github.com/thephpleague/flysystem) — by Frank de Jonge / PHPLeague.
* [Gaufrette](https://github.com/KnpLabs/Gaufrette) — from KNP Labs

### Plugins for CMS

Hello WordPress users, there are many plugins, [WP Offload S3](https://wordpress.org/plugins/amazon-s3-and-cloudfront/), [W3 Total Cache](https://wordpress.org/plugins/w3-total-cache/) for you that you can use to upload your files directly somewhere else.


## Composer is standard

With Old Apps you had to trigger the dependency manager [Composer](composer). Now with the New Apps Composer is a standard part of the deployment. As long as your repository contains a `composer.lock` and `composer.json` file (on top level), a `composer install` will be executed on every `git push`. The first Composer installation can take some time (depending on the amount of packages you are using). Any subsequent deployment will re-use the vendor folder and is thereby blazing fast.

## New deployment file version

You can fine-tune your build with additional scripts and variables by creating a file called `fortrabbit.yml` on top-level of your repository. Old Apps are using the [deployment file version 1](deployment-file-v1-old-app), New Apps are using [deployment file version 2](deployment-file-v2).



## No Git submodules

Git submodules are not supported by New Apps. But why should you use them anyway when there are [subtrees](http://blogs.atlassian.com/2013/05/alternatives-to-git-submodule-git-subtree/)? For lazy shell users, there is also [stree](https://github.com/tdd/git-stree), which reduces the amount of typing by quite a lot.

## HTTP-Auth without htaccess directive

New Apps don't require you to write a [.htaccess directive](http-auth) any more.


## App repository reset

Only for New Apps: To start with a complete new Git history, you can now reset your repository. This can be done with the `reset` command like so:

```bash
ssh git@deploy.eu2.frbit.com reset your-app-name
```

The reset operation is non-destructive, meaning: It does not generate a release. Thereby your live App continues to operate without any interruption. The new release will only be build on the next push (of your new code base). A repository reset also removes any sustained directory (the `vendor` folder, so a following [Composer](composer) install requires to download everything once again).


## Private repositories

Old Apps offered a keygen hook to realize access to [private Composer repos](private-composer-repos). With the New Apps you can create a new SSH keypair for your App, using the `keygen` command:

```bash
ssh git@deploy.eu2.frbit.com keygen your-app-name
```

This will print out a public key, which you can then install with your private repository (on Github, Bitbucket or wherever).


## Simplified branching for multi-staging setups

Usually, only the remote Git `master` branch will be deployed. With the New Apps you can also create a branch with the same name as your App, which will be preferred over the master branch. That way it is easier to set up a [multi-staging environment](multi-staging).


## New streaming log access

With the Old Apps you can access the logs via SSH. New Apps don't offer direct file access (nor historical logs) but you can tail a stream of live logs.

### Sources

```bash
# Per default all sources are tailed together:
ssh log@log.eu2.frbit.com tail app-name

# Only Apache access log, all incoming requests with response status, time-stamp, additional headers and the first line of the request:
ssh log@log.eu2.frbit.com tail app-name source:apache_access

# Only Apache error log, which can be very helpful to debug `.htaccess` files or the like:
ssh log@log.eu2.frbit.com tail app-name source:apache_error

# Only PHP error logs, which contain whatever your App writes on `error_log()`:
ssh log@log.eu2.frbit.com tail app-name source:web_php_error

# Only web standard error output, which containins everything written by your App to `STDERR`:
ssh log@log.eu2.frbit.com tail app-name source:web_stderr
```

### Other parameters

```bash
# The `source` parameter can be use multiple times:
ssh log@log.eu2.frbit.com tail app-name source:apache_access source:apache_error

# Use the `mono` parameter to disable output colors:
ssh log@log.eu2.frbit.com tail app-name mono
```


## Further readings 

We have also written a comprehensive [blog post](http://blog.fortrabbit.com/new-apps-are-here) about the features, benefits, limits and backgrounds of the New Apps.
