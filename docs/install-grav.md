---

template:   article
title:      Install Grav
naviTitle:  Grav
lead:       Grav is a new PHP CMS based on Twig & Markdown. Learn how to install and tune Grav on fortrabbit.
tags:
    - php
    - install

---

[Grav](http://getgrav.org) runs pretty much out of the box, as long as you use the [zip package installation](http://learn.getgrav.org/basics/installation#option-1-install-from-zip-package) installation method and not the [GitHub installation](http://learn.getgrav.org/basics/installation#option-2-install-from-github) method.

Install
-------

We assume you've already created an [App](app) with fortrabbit. You also need a local Grav installation. You can either use an existing or a new one. This guide explains you how to start from scratch:

First [download](http://getgrav.org/downloads) and unpack the latest Grav locally. It unpacks in the subfolder `grav`. When it's done, change into that folder and set it up with your Git remote on fortrabbit

```bash
$ cd grav
$ git init .
$ git remote add fortrabbit git@deploy.eu2.frbit.com:your-app.git
```

Now, before commiting anything, you should exclude the `vendor/` and the `cache/` folder. Create the file `.gitignore` with the following contents:

```
vendor
cache/*
!cache/.gitkeep
```

Now you can add everything and push everything to your App:

```bash
$ git add -A
$ git commit -m 'Initial'
$ git push -u fortrabbit master
```

Done. Your Grav site is now online and you can visit it in your browser!

Tuning
------

Word: The above guide is just the quickest and most basic way to get things up. Please mind that there are various ways to install and manage Grav. Apart from downloading you can: use Composer, clone the Git repo directly and also use the Grav CLI. Advanced users might also consider to have separated repos for the Grav core, Plugins, Themes and the actual content in pages.



### Install theme

Grav supports themes and plugins, which both can be installed with the `bin/gpm` tool. Assuming you followed the [zip package based installation](http://learn.getgrav.org/basics/installation#option-1-install-from-zip-package), as shown above, installing plugins or themes is easy: Find the [theme you like](http://getgrav.org/downloads/themes), then install it as usual. Following an example. Run locally:

```bash
$ bin/gpm install soraarticle
```

Open `user/config/system.yaml` and set `pages/theme` to `soraarticle`. Now commit and push:

```bash
$ git add -A
$ git push -m 'New theme'
$ git push
```

Done.

### Install plugin

Find the [plugin you need](http://getgrav.org/downloads/plugins) then install it as usual (we use download here). Following an an example. Run locally:

```bash
$ php bin/gpm install simplesearch
```

Then configure the plugin, if needed (usually in `user/plugin/<pluginname>/<pluginname>.yaml`), commit and push:

```bash
$ git add -A
$ git push -m 'New plugin'
$ git push
```

Done.


### Grav Admin VS ephemeral storage

The popular [Grav Admin plugin](https://learn.getgrav.org/admin-panel/introduction) provides a way to manage a site in a guided web interface similar to the "wp-admin". This is great, when you want your client to manage her/his Grav site her/himself. Unfortunately it does not play well with ephemeral storage. Like with other PaaS — the local file system storage of the App will be wiped on each deploy — think 12-factor-app. So all local changes made in the Grav Admin panel will be gone.

Currently you should: use Grav on fortrabbit for your own private projects where you: write locally in your text editor, handle the pages in Git.