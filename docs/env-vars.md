---

template:   article
title:      Using environment variables in PHP
naviTitle:  Environment variables
lead:       ENV vars help to create and shape the environment of where the code runs.

keywords:
    - ENV vars
    - Environment variables

seeAlsoLinks:
    - secrets
    - multi-staging

tags:
    - php
    - beginner

---

## Problem

You probably run at least two deployments of your App: a local one for development and one in the cloud (here on fortrabbit) for production. Both instances probably have access to a database. Your local MySQL has of course different credentials as the remote one. Your config file storing this information is under Git version control. So how to deal with different environment-specific configurations of one App?

## Solution

Use environment variables to keep local configurations out of the main code base.

## ENV vars in PHP

There are two global variables called `$_SERVER` and `$_ENV` which contain environment variables that are available through the operating system and the PHP server. Within fortrabbit you should use `$_SERVER`, eg:

```php
echo $_SERVER["MY_ENV_VAR"];
```

## Adding ENV vars

You can enter ENV vars on fortrabbit in the [Dashboard](dashboard). You'll do so in the settings of your [App](app). You can edit ENV vars one by one or import multiple at once.

## ENV vars vs security

**Note**: Before diving into how to securely use ENV vars, please check out the [App secrets](secrets), which provide already an out-of-the-box solution.

Storing credentials (passwords, secrets, ..) in environment variables is not without risk. They can be exposed, due to programming errors or oversights. Please read an BLOG[in-depth discussion in our Blog](how-to-keep-a-secret).

Our proposed solution is to encrypt environment variables when storing them in the Dashboard and decrypting them whenever accessed. This way accidentally exposed environment variable contents do not pose a risk nor does your code (in the Git history) contain any credentials or a method to derive them.

Solutions for specific frameworks or CMS can be found in their respective [installer guides](/#install). A generic approach would be to PHP's `mcrypt` extension as following:

### Generic ENV var encryption

Create a secret key and use the `mcrypt_encrypt()` function to encrypt and then base64 encode your variables. An exemplary PHP script doing that can be [downloaded from this Gist](https://gist.github.com/ukautz/3573878af39e81c009fa) and then executed locally.

Start with generating a new key and storing this key somewhere in your app's bootstrap code:

```bash
php encrypt.php genkey
# QEbfdTYNH23YddNUa1srixRAwBVs2L5p
```

Then you can encrypt the values of each environment variable, you want to store safely:

```bash
php encrypt.php enc "The Key" "Some Value"
# ENC:YToyOntp...t8CI7fQ==
php encrypt.php enc "The Key" "Some Other Value"
# ENC:XioptiwF...tmMNs9p==
```

To access the decrypted values you simply do the reverse: decode from base64 and decrypt using `mcrypt_decrypt()`. Again, here is an [exemplary Gist](https://gist.github.com/ukautz/0b430aafc7cc996fc946). Using it, you can then decode the environment variables in your app's bootrap:

```php
// ..
bootstrapSecEnv("Your Key");
// ..
```

Later, when you need to access your now decrypted environment variables:

```php
// ..
$dbUser = secEnv("DB_USER");
$dbPass = secEnv("DB_PASS");
// ..
```
