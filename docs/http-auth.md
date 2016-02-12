---

template:   article
title:      Using Basic access authentication aka HTTP Auth on fortrabbit
naviTitle:  HTTP Auth
lead:       How to set up your App in a way that the browser prompts for username/password.

keywords:
    - HTTP Auth
    - password for website

externalLinks:
    - http://en.wikipedia.org/wiki/Basic_access_authentication

seeAlsoLinks:
    - quirks


tags:
    - php

---

## Problem

You probably don't want the whole world to see your development in progress. So you want to restrict access to a fortunate few using [HTTP (basic) auth](https://en.wikipedia.org/wiki/Basic_access_authentication). In the fortrabbit PHP/FPM infrastructure neither `PHP_AUTH_USER` nor `PHP_AUTH_PW` are available - but you can hack around easily.

## Solution

To utilize HTTP (basic) Auth, you need to add a directive in your `.htaccess` file, forwarding the `Authorization` header as an environment variable. This variable then contains the base64 encoded authentication data, which you then can then decode to the `PHP_AUTH_USER` and `PHP_AUTH_PW`.

### Modify .htaccess file

```
# ..
RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=REMOTE_USER:%{HTTP:Authorization}]
```

### Decode auth header in PHP

```php

// header was not provided
if (empty($_SERVER['REMOTE_USER'])) {
    header('WWW-Authenticate: Basic realm="My Realm"');
    header('HTTP/1.0 401 Unauthorized');
    echo 'Need auth!';
    exit;
}

// extract user and pw from encoded auth data
list($_SERVER['PHP_AUTH_USER'], $_SERVER['PHP_AUTH_PW']) = explode(
    ':',
    base64_decode(substr($_SERVER['REMOTE_USER'], 6))
);
```

