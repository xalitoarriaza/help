---

template:   article
title:      All about Memcache
naviTitle:  Memcache
lead:       Speed speed speed. Why and how to do caching with Memcache on fortrabbit.
linkOther:  /memcache-old-app

keywords:
   - memcached
   - memcache

seeAlsoLinks:
   - app-design


tags:
   - cache

---


## Problem

[Caching](best-practices#toc-prepare-to-cache) is a cornerstone of the "snappy App". As soon as your App spreads across more than one PHP Node you probably need a network cache. Otherwise, user sessions would not work.

## Solution

Meet Memcache(d) caching: the standard network cache which is extremely fast and reliable. Applying it correctly can boost the performance of most web applications dramatically. We offer optional Memcache scalings, which provision dedicated memcached servers, running on a single or two Nodes for data redundancy.

The foremost use-case example are user sessions in the aforementioned multi-node scenario. Aside from those any kind of buffered data - such as expensive MySQL query results - can be stored and received extremely fast.


## Usage

You can enable and scale Memcache in the [Dashboard](dashboard). The smallest available scaling is a memcached server running on a single Node and is primarily recommended for development scenarios. The additional Production scalings grow in size of memory and span across two nodes to offer high availability in outage scenarios.

In PHP you have two drivers to connect to a memcached server: [Memcache](http://php.net/manual/en/book.memcache.php) and [Memcached](http://php.net/manual/en/book.memcached.php). If in doubt, use the former one.

## Production PHP and Memcache

All PHP production scaling are running on two Nodes. If your App needs to handle session data, you need to keep it consistent across the Nodes. Memcache is also our recommended a solution for this. See our [PHP scaling article](php-scaling#toc-memcache-is-probably-needed).


## Scaling

In the Dashboard, go to your App and click on Memcache under the scaling options. Please also see the [Memcache scaling](scaling#toc-memcache) section. 

Using a plan with two Nodes allow you two modes:

1. **Combined**: data is stored only on one of the nodes > twice the memory size
2. **Redundant**: data is stored on both nodes > one nodes can fail

The former mode is not recommended. So here is how you enable redundant mode when using the `Memcached` library:

```php
$secrets = json_decode(file_get_contents($_SERVER["APP_SECRETS"]), true);
$mc      = $secrets['MEMCACHE'];

// enable redundant session handler
ini_set('session.save_handler', 'memcached');
ini_set('memcached.sess_number_of_replicas', 1);
ini_set('memcached.sess_consistent_hash', 1);
ini_set('memcached.sess_binary', 1);
ini_set('session.save_path', implode(", ", [
	"{$mc['HOST1']}:{$mc['PORT1']}",
	"{$mc['HOST2']}:{$mc['PORT2']}"
]));

session_start();

// use redundant memcache for user data
$m = new Memcached();
$m->addServer($mc['HOST1'], $mc['PORT1']);
$m->addServer($mc['HOST2'], $mc['PORT2']);
$m->set('SomeKey', 123);
$m->get('SomeKey');
```


## Alternatives

There are some other cache stores, most importantly [Redis](http://redis.io/). We currently don't offer any of those directly, but you can easily use a [3rd party provided](external-services#toc-application-helpers) with your fortrabbit App.
