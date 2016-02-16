---

template:      article
title:         All about Domains & DNS
naviTitle:     Domains
lead:          The world of App URLs, domains and DNS. How to add and manage custom domains
linkOther:     /domains-old-app


tags:
    - beginner

keywords:
    - TLD
    - Top Level Domain
    - registration
    - ordering

seeAlsoLinks:
    - directory-structure
    - tls

---

Each fortrabbit App has its own, unique App URL. Additionally you can route any external domain to your App. Learn all about DNS and domains settings on fortrabbit.


**MIND**: fortrabbit is not offering direct domain ordering like in classical hosting. You have to register your domain elsewhere. Take care that your external domain [DNS](external-services#toc-dns-as-a-service) or [registration](external-services#toc-domain-registration) service supports forwards (ALIAS or ANAME records).

## App URL

While creating your fortrabbit App you will be asked for an App name. This name can't be changed later on and is used in many places as an identifier. The most prominent one is your default App URL which looks like this: `http://my-app.frb.io/`. This is where you can always reach your App thru the web browser. Use the App URL for development, testing and to connect to external services.

## Route a custom domain

Many different and advanced configurations are possible, see below. Here is the dead simple setup:

#### External provider

1. Point `www.yourproject.com` using a `CNAME` record to your fortrabbit App hostname `my-app.frb.io`;
2. forward your naked domain `yourproject.com` to the actual subdomain `www.yourproject.com`.

PATIENCE YOUNG JEDI: Most Domain DNS records have a TTL (Time To Live) of 24 hours. This means that all name servers will only look every 24 hours if a domain target has been changed. In some cases it might even take longer.

#### fortrabbit dashboard

1. Enter the domain `www.yourproject.com` as an external domain in the domain settings of the App;
2. repeat, if you need, with `yourproject.com`.

## Set a custom root path

Per default the domains of the App will route to the `~/htdocs` [folder](directory-structure). In some cases, you need to set a different document root or want to route different domains to different folder. Laravel, for examples, requieres you to use `~/htdocs/public` per default.

You can do this by writing the relative path to the sub-folder (all folders below the htdocs folder are allowed) in the Root Path field right to the domain URL.

For example, if you want to use the folder `~/htdocs/web`, just enter `web` in the input. If you want to use the folder `~/htdocs/app/webbroot`, just enter `app/webroot`.

## Change the default domain

This is an optional setting. Per default your App URL is the default domain. You can change this in the dashboard — so links and the thumbnail preview generation will work with the new primary domain.

## Advanced routing alternatives

The world of DNS is one of its own. Let's dive into it so you can understand the backgrounds and explore alternative settings.

### Non naked domain

`www.fortrabbit.com` — Although this looks a bit antiquated, it's the modern "cloud enabled" way to go.

Back in the days the www. prefix indicated that's the address to type in the browser. Nowadays the www. prefix indicates I am ready to move this App in seconds to another server location. Test it: all big players run on a www. subdomain. The name of the subdomain prefix is not so important, but `www` is the convention for (marketing) entry points.

Non naked domains are usually routed thru CNAME records.

### Naked domain

`fortrabbit.com` — Indeed this looks visually more appealing, cleaner, leaner than his ugly brother the non-naked domain. Unfortunately this most likely means that the domain is routed via A-Record, which means an IP address is in place. In the world of cloud computing this is not smart, you (in this case we do it for you) want to be able to move your App around quickly, to scale up and down for instance, or in case of an emergency (dDos attack for example).

#### Problem: CNAME vs naked domains

It is possible to create a CNAME record for a naked domain. The problem is that `CNAME` records do not behave like the other records (`A`, `MX`, `TXT`, …). They are *greedy*. This means: They *overwrite* all other records.

An example: assume you have a domain `domain.tld` and want to receive mails for it. So you create an `MX` record. Say it points to `mail.domain.tld`. If you now would create a `CNAME` record directly for `domain.tld` pointing to `otherdomain.tld`, every lookup of any record for `domain.tld` will be made on `otherdomain.tld` instead (aside from the `CNAME` record itself).

#### Solution 1: Use ANAME/ALIAS provider

There are DNS providers, which allow you to use non-standard records, which they call `ANAME` or `ALIAS`. Those combine the *positive* attributes of `CNAME`  with `A` records. In short: you can route a naked domain to a hostname without loosing your `MX` records.

Here is a [list of providers](/external-services#toc-dns-as-a-service).

#### Solution 2: Use forwarding

Many domain providers support a simple HTTP redirect. This basically means: they provide a web server for you and redirect all incoming requests to `http://domain.tld/` to `http://www.domain.tld/`.

Alternative names for this feature are *Domain forwarding*, *Web forwarding*, *Domain masking* and the like.

**Documentation of some providers:**

* [GoDaddy](https://support.godaddy.com/help/article/422/manually-forwarding-or-masking-your-domain-name)
* [Gandi](https://wiki.gandi.net/en/domains/management/domain-as-website/forwarding)
* [1&1](http://help.1and1.com/domains-c36931/manage-domains-c79822/domain-destination-c38672redirectforward-your-domain-a594868.html)

**Alternative:** If your domain provider does not support forwarding, you can use a [forwarding service](/external-services#toc-domain-forwarding-as-a-service).

#### Solution 3: Use CloudFlare

If you use CloudFlare, you solved the problem already, since they only route to URLs anyway.

### Wildcards

You probably want to route all requests for subdomains to your fortrabbit App like so: `*.mydomain.com` — for instance when the users of your App create spaces within your domain name. That's possible, but for security reasons we'll need to verify your request. Also after you have a setup a wildcard for a domain, all other new requests for custom routings for this domain also need to be verified.
