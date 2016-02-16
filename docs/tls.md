---

template:      article
title:         Using the TLS component
naviTitle:     TLS/SSL/HTTPS
lead:          "HTTPS for the rest of us! How to use Secure Socket Layer connections for custom domains on fortrabbit. Setting up TLS is (still) a bit of a hassle, but together we will make it! Here are all the infos you'll need."
linkOther:     /ssl-old-app

keywords:
    - ssl
    - addon
    - add-on
    - domain
    - naked
    - subdomain

seeAlsoLinks:
    - domains
    - external-services

tags:
    - DNS

---

## Problem

The interwebs is full of criminals trying to read your communication.

## Solution

Encrypt the connection between your App and your users with a Transport Layer Security (TLS) connection aka `HTTPS`.

You already can use a piggyback `HTTPS` on your App URL. But what about your own [custom domain](domains)?

The fortrabbit TLS Component enables you to establish a verified HTTPS-connection on a [custom domain](domains). The TLS Component itself needs to run separated, that's why there is a dedicated load balancer hosting your SSL/TLS certificate and that's also why we have to charge for it.

### TLS, SSL, HTTPS, WTF?

Transport Layer Security (TLS) is the successor to the Secure Sockets Layer (SSL), it's a protocol to ensure privacy between applications and users. HTTPS is the Secure version of the Hyper Text Transfer Protocol. It's used between browser and server.


## Usage

Let's be honest here. Although TLS is one of the usual business requirements, it's still a bit complicated to configure. We would love to provide a more convenient solution. Fact is that most of the stuff here is beyond our control, so that's the way it is now:

1. have an [external domain](external-services#toc-domains) registered
2. create a key and a cert locally
3. purchase an SSL/TLS cert from an [external provider](external-services#toc-ssl-certificates)
4. book the TLS component for your App in the Dashboard
5. upload your key and cert(s) to the Dashboard
5. route all your (sub)domains to your App's hostname via CNAME


### Create a new key and certificate request

Your external SSL/TLS certificate provider will ask you for those. To create a new key and a certificate signing request (CSR), issue the following commands on the console of your local machine (Mac/Linux):

```bash
openssl req -new -nodes -keyout my-app.key -out my-app.csr -newkey rsa:2048
# Generating a 2048 bit RSA private key
# ..........................................................................................++
# .............................................+++
# writing new private key to 'my-app.key'
# -----
# You are about to be asked to enter information that will be incorporated
# into your certificate request.
# What you are about to enter is what is called a Distinguished Name or a DN.
# There are quite a few fields but you can leave some blank
# For some fields there will be a default value,
# If you enter '.', the field will be left blank.
# -----
# Country Name (2 letter code) [AU]:DE
# State or Province Name (full name) [Some-State]:Berlin
# Locality Name (eg, city) []:Berlin
# Organization Name (eg, company) [Internet Widgits Pty Ltd]:fortrabbit
# Organizational Unit Name (eg, section) []:Website
# Common Name (e.g. server FQDN or YOUR name) []:www.fortrabbit.com
# Email Address []:info@fortrabbit.com
# -----
# Please enter the following 'extra' attributes
# to be sent with your certificate request
# A challenge password []:
# An optional company name []

# Update key format
openssl rsa -in my-app.key -out my-app.rsa.key
```

Do not enter a password! Also: if you plan on using `www.yourdomain.tld`, don't miss the `www.` in the "Common Name"!

With the now generated CSR, you can go to an external [certificate vendor](external-services#toc-ssl-certificates), which will issue a certificate for you.


### Convert existing key to RSA format

If you already have a private key and it begins with `----BEGIN PRIVATE KEY----` you need to convert it to the RSA format, using:

```bash
openssl rsa -in my-app.key -out my-app.rsa.key
# writing RSA key
```

The RSA format is required by our TLS implmentation.


### Book & install

Now you are ready to book the TLS component in the fortrabbit Dashboard. During the booking you will be asked to insert your certificate, your private key and possibly your intermediate (chain) certificates, which your certificate vendor might have send you.

### Route domains

Whether you want to use HTTP or HTTPS you need to [route all your (sub)domains](domains#toc-route-a-custom-domain) to your App's hostname.

## Common issues

### Solve SSL verification errors

This is the fix for verification errors when using ssl_verify. If you are receiving an error like the following:

```
error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
```
You need to set the `capath` as well. Assuming you are using PHP streams, it would look like this:
```php
$context = stream_context_create();
stream_context_set_option($context, 'ssl', 'verify_peer', true);
stream_context_set_option($context, 'ssl', 'capath', '/etc/ssl/certs'); # <<< that's the one
stream_context_set_option($context, 'ssl', 'allow_self_signed', false);

$fp = stream_socket_client("thedomain.tld:443", $errno, $errstr, 5, STREAM_CLIENT_CONNECT, $context);
# ..
if (stream_socket_enable_crypto($fp, true, STREAM_CRYPTO_METHOD_SSLv3_CLIENT) === false) {
    die("Failed to verify certificate");
}
#...
```

For cURL, the option CURLOPT_CAPATH needs to be set to `/etc/ssl/certs`.


### Redirect all requests to HTTPS

Create or modify an `.htaccess` file in your document root folder like so:

```
RewriteEngine On
RewriteCond %{SERVER_PORT} 80
RewriteRule (.*) HTTPS://%{HTTP_HOST}/$1 [R=301,L]
```

The reverse (redirect all traffic to HTTP) would be:

```
RewriteEngine On
RewriteCond %{SERVER_PORT} 443
RewriteRule (.*) http://%{HTTP_HOST}/$1 [R=301,L]
```

### Get help

Unsure about certain parts? Have questions? Yes it is confusing. Don't be afraid to ask. We are experienced and we can help you get this done. [Contact us](http://www.fortrabbit.com/contact)!


### Let's encrypt

There is a new service called [let's encrypt](https://letsencrypt.org/) to offer a free Certificate Authority and beat tools around it. There is no support on fortrabbit yet.