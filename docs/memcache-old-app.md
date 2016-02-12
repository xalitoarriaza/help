---

template:   article
title:      Memcache on fortrabbit
naviTitle:  Memcache
lead:       Speed speed speed - caching on fortrabbit.
dontList:   true
oldApp:     true
linkOther:  /memcache

keywords:
   - memcached
   - memcache

seeAlsoLinks:
   - app-design


tags:
   - cache

---


## Problem

[Caching](best-practices#toc-prepare-to-cache) is a cornerstone of the "snappy App". APC is nice, but sometimes you just need more.

## Solution

Meet Memcache(d) caching: the standard network cache. Applying it correctly can boost the performance of most web applications dramatically.

We offer optional Memcache plans, which provision dedicate memcached servers, running on shared nodes, usable as data caches for the web applications. They give the developer the opportunity to buffer once calculated data (a database query result) in the memory and access the cached data instead of recalculating it every time. As the data is stored in memory and transported over a fast network, the access is extremely fast.

Apps with PHP on one single node can, but do not need to implement Memcache. Apps with PHP running on more than a single node require memcache for many use cases, as it is important to access the same data from all the nodes.


## Usage

You can enable and scale Memcache in the Dashboard. The smallest available scaling is a memcached server running on a single node and is primarily recommended for development scenarios. The additional plans grow in size of memory and span across two nodes to offer a higher availability in outage scenarios.



### Redundant mode

Here is how you enable redundant mode for the `Memcache` library:

```php
// enable redundant session handler
ini_set('memcache.session_redundancy', 3);
ini_set('session.save_handler', 'memcache');
ini_set('session.save_path', 'tcp://memcachecluster.frbit.com:11211, tcp://memcachecluster.frbit.com:11212');
session_start();

// use redundant memcache for user data
ini_set('memcache.redundancy', 2);
$m = new Memcache();
$m->addServer('memcachecluster.frbit.com', 11211, false);
$m->addServer('memcachecluster.frbit.com', 11212, false);
$m->set('SomeKey', 123);
$m->get('SomeKey');
```

Yes, `memcache.session_redundancy` needs [really](https://bugs.php.net/bug.php?id=58585) to be set to `3`! We think this is weird, too.


