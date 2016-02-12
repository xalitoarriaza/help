---

template:  article
title:     Install Phalcon 2
naviTitle: Phalcon 2
lead:      Looking for sPHPeed? Here you learn how to best getting started with Phalcon 2 on fortrabbit.
linkOther: /install-phalcon-old-app

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

Phalcon uses `public` as doc root, you need change that in the fortrabbit Dashboard under for your Apps [Domains]().

### Enable Phalcon extension

As Phalcon is an C Extension, you need to enable it in the Dashboard under your Apps [PHP Extensions](php-extensions#toc-frameworks).

