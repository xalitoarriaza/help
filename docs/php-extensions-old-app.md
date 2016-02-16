---

template:   article
title:      Manage PHP extensions
naviTitle:  PHP extensions
lead:       See which PHP extensions and options are available on fortrabbit.
tags:
     - advanced
dontList:      true
oldApp:        true
linkOther:     /php-extensions

seeAlsoLinks:
     - app

---

PHP delivers a wide range of functionality. However, something is always missing and a language should not be overcrowded with special case solving methods. Still, sometimes it needs to be written in C due to performance or requirements to link to native system libraries. Therefore there are extensions.


## Upgrade policy

Bugs and flaws in source code are a major reason for security breaches, so keeping up-2-date with as many as possible components is essential. We subscribe to an ASAP policy of implementing security patches to protect our customers from malicious attempts to the best of our capabilities.

A side effect of providing the latest stable release of all components is that they come sometimes packaged with API changes. Extensions like YAML, MCrypt or alike probably won't be the source of any trouble as they are already quite mature and integral API changes are not to be expected. However, some "younger" extensions, especially in Alpha or Beta stage, might bring core changes along with patches for security holes.

You should be aware of this and use Alpha or Beta grade extensions with caution.

## Configurable extensions

#### core

This is not a real extension but some configurable PHP core (and built-in extension) settings.



#### apc

The [Advanced PHP Cache](http://php.net/manual/en/book.apc.php) implements a fast opcode cache.

**[apc.stat](http://php.net/manual/en/apc.configuration.php)** If enabled (1) changed files will be loaded from disk automatically. If disabled (0), cached files will always be delivered from cached. Disabling can be much faster, if many PHP files are used (via require or include), but no changes on PHP files will be applied until APC is flushed.

**[apc.ttl](http://php.net/manual/en/apc.configuration.php)** Time to live for system cache entries.

**[apc.user_ttl](http://php.net/manual/en/apc.configuration.php)** Time to live for user cache entries.

**[apc.gc_ttl](http://php.net/manual/en/apc.configuration.php)** Time to live on the garbage-collection list.

### Drivers






#### mysqlnd

The [MySQL native driver](http://php.net/manual/en/book.mysqlnd.php) replaced the old `libmysql` based PHP MySQL driver in PHP 5.3.0.

#### pgsql

Driver for [PostgreSQL](http://php.net/manual/en/book.pgsql.php) databases. In order to access an external PostgreSQL database you need to request a [firewall white-listing](security#toc-firewalling).

#### redis

Driver for [Redis](https://github.com/nicolasff/phpredis) key/value store.

#### mongo

Driver for [MongoDB](http://php.net/manual/en/book.mongo.php) databases. Please request [firewall white-listing](security#toc-firewalling) if you need to access an external MongoDB database with a port differing from the standard port (27011).




#### sqlite3

Driver for [SQLite](http://php.net/manual/en/book.sqlite.php) databases.

PLEASE NOTE: We are using a shared network storage. If your App runs on more than one Node, SQLite database usage is NOT recommended.

#### memcache

One of the two drivers for [memcached](http://php.net/manual/en/book.memcache.php). We also offer a [Memcache Component](memcache) .

#### memcached

The other driver for memcached.

#### stomp

The [Stomp Client](http://php.net/manual/en/book.stomp.php) extension allows PHP applications to communicate with any Stomp compliant Message Brokers through easy object oriented and procedural interfaces. Please request a [firewall white-listing](security#toc-firewalling) if you need to access an external Stomp broker.

### Frameworks





#### phalcon

[Phalcon](http://phalconphp.com/) is an extremely fast, new-school PHP Framework written in C.





#### yaf

[YAF](http://php.net/manual/en/book.yaf.php) is also an extremely fast PHP Framework based on C available since PHP 5.2.

### Code protection

#### sourceguardian

[SourceGuardian](http://www.sourceguardian.com/) protects source code and can be used to enforce license policies.

#### ioncube

[ionCube](http://www.ioncube.com/) protects source code and can be used to enforce license policies.

### Debugging





#### xdebug

The [XDebug](http://xdebug.org/docs/) extension is the standard helper for debugging PHP code. Provides stack and function traces, infos about memory allocation and protects against infinite recursions. Also a profiler for deep code and execution analysis can be used.

**[xdebug.profiler_enable_trigger](http://xdebug.org/docs/all_settings#profiler_enable_trigger)** If enabled (1), you can use GET or POST parameter XDEBUG_PROFILE to trigger writing profiler data in `~/data` directory using the file specified in `profiler_oputput_name`.

**[xdebug.profiler_output_name](http://xdebug.org/docs/all_settings#profiler_output_nameFile)** name template for outputting profiler data.






#### xhprof

The [XHProf](http://php.net/manual/en/book.xhprof.php) extension is a light-weight hierarchical profiler.

### Tools

#### curl

The [CURL](http://php.net/manual/en/book.curl.php) extension allows you to connect and communicate to many different types of servers with many different types of protocols.

#### gd

The [GD](http://php.net/manual/en/book.image.php) extension allows you to create and manipulate image files in a variety of different image formats.

#### geoip

The [GeoIP](http://php.net/manual/en/book.geoip.php) extension allows you to find the location of an IP address.

#### gmp

The [GNU Multiple Precision](http://php.net/manual/en/book.gmp.php) extension implements arbitrary precision arithmetic, operating on signed integers, rational numbers, and floating point numbers.

#### http

The [HTTP](http://php.net/manual/en/book.http.php) extension eases handling of HTTP URLs, dates, redirects, headers and messages in a HTTP context.

#### igbinary

The [Igbinary](https://github.com/igbinary/igbinary) extension is a drop in replacement for the standard PHP serializer.

#### imagick

The [ImageMagick](http://php.net/manual/en/book.imagick.php) extension is a native PHP extension to create and modify images using the ImageMagick API.

#### imap

The [IMAP](http://php.net/manual/en/book.imap.php) extension enables you to operate with the IMAP protocol, as well as the NNTP, POP3 and local mailbox access methods.

#### intl

The [Internationalization](http://php.net/manual/en/book.intl.php) extension is a wrapper for [ICU](http://www.icu-project.org) library, enabling you to perform [UCA](http://www.unicode.org/reports/tr10/-conformant) collation and date/time/number/currency formatting.

#### mcrypt

The [Mcrypt](http://php.net/manual/en/book.mcrypt.php) extension is an interface to the mcrypt library, which supports a wide variety of block algorithms such as DES, TripleDES, Blowfish (default), 3-WAY, SAFER-SK64, SAFER-SK128, TWOFISH, TEA, RC2 and GOST in CBC, OFB, CFB and ECB cipher modes.

#### oauth

The [OAuth](http://php.net/manual/en/book.oauth.php) extension provides OAuth consumer and provider bindings.

#### runkit

The [runkit](http://php.net/manual/en/book.runkit.php) extension provides means to modify constants, user-defined functions, and user-defined classes.

#### ssh2

The [ssh2](http://php.net/manual/en/book.ssh2.php) extensions provides access to resources (shell, remote exec, tunneling, file transfer) on a remote machine using a secure cryptographic transport.

#### tidy

The [Tidy](http://php.net/manual/en/book.tidy.php) extension is a binding for the Tidy HTML clean and repair utility which allows you to not only clean and otherwise manipulate HTML documents, but also traverse the document tree.

#### xsl

The [XSL](http://php.net/manual/en/book.xsl.php) implements the XSL standard, performing XSLT transformations using the libxslt library.

#### yaml

The [YAML Data Serialization](http://php.net/manual/en/book.yaml.php) extension implements the [YAML Ain't Markup Language](http://www.yaml.org/) data serialization standard.