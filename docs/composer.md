---

template:   article
title:      Leveraging Composer
naviTitle:  Composer
lead:       Learn how to integrate Composer into your development workflow with fortrabbit.
linkOther:  /composer-old-app

seeAlsoLinks:
   - deployment

tags:
   - git

keywords:
  - composerphp
  - getcomposer
  - Dependency Manager

---

No need to explain this anymore: [Composer](http://getcomposer.org) is the defacto standard to handle PHP application dependencies, as well as providing mechanisms to keep them up2date.

## Usage

**TL;DR**: Just `git push`. On fortrabbit you [(of course) exclude](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md) the `vendor` folder from Git. This directory is created by Composer within in your project locally and contains all the packages you are using. This way only your own code — not any dependency — is included into Git.

During deployment to fortrabbit Composer will be executed automatically, assuming that both the `composer.lock` file and the `composer.json` file are present.

The deployment build process executes `composer install` and never the update variant to make sure that exactly the packages your App requires are installed.

### Composer in the deployment file

You can fine tune your deployment behavior and aspects of Composer in the deployment file. See the options withunder the `composer:` keyword [here](deployment-file-v2).

### Use Private repositories in Composer

You need to add the private repositories into your `composer.json` file? Read on [here](private-composer-repos).
