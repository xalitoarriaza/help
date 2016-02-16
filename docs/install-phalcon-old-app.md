---

template:  article
title:     Install Phalcon - for Old App
naviTitle: Phalcon
lead:      sPHPeeeeed! Here you learn how to best getting started with Phalcon on fortrabbit.
dontList:  true
oldApp:    true
linkOther: /install-phalcon

keywords:
    - phalconphp
    - framework
    - php extension
    - c extension
    - hhvm


tags:
    - php
    - install


---

[Phalcon](http://phalconphp.com/en/) is a web framework delivered as C extension providing high performance and low resource consumption.

## Usage

We recommend our [Git deployment](git) workflow to install and use Phalcon. Here are some noticeable details to get it right:

### Set Phalcon document root

Phalcon uses `public` as doc root, you need change that in the fortrabbit Dashboard under your Apps [Domains]().

### Enable Phalcon extension

As Phalcon is a C Extension, you need to enable it in the Dashboard under your Apps [PHP Extensions](php-extensions#toc-frameworks).

### Use Phalcon developer tools

Here is a quick example how to:

```bash
cd ~/htdocs

# clone the dev tools
git clone https://github.com/phalcon/phalcon-devtools
# output


# run the tools
cd ~/htdocs
php phalcon-devtools/phalcon.php list

Phalcon DevTools (1.0.1)

Available commands:
  commands (alias of: list, enumerate)
  controller (alias of: create-controller)
  model (alias of: create-model)
  all-models (alias of: create-all-models)
  project (alias of: create-project)
  scaffold
  scaffold-dbm
  migration
  webtools
```