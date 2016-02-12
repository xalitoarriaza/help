---

template:      article
title:         All about Domains & DNS - Old App
naviTitle:     Domains - Old App
lead:          The world of App URLs, domains and DNS.
dontList:      true
oldApp:        true
linkOther:     /domains

tags:
    - beginner

keywords:
    - TLD
    - Top Level Domain
    - registration
    - ordering

seeAlsoLinks:
    - directory-structure

---

Each fortrabbit App has it's own, unique App URL. Additionally you can route any external domain to your App. Learn all about DNS and domains settings on fortrabbit.


**MIND**: fortrabbit is not offering direct domain ordering like in classical hosting. You have to register your domain elsewhere. Take care that your external domain [DNS](external-services#toc-dns-as-a-service) or [registration](external-services#toc-domain-registration) service supports forwards (ALIAS or ANAME records).

## App URL

While creating your fortrabbit App you will be asked for an App name. This name can't be changed later on and is used in many places as an identifier. The most prominent one is your default App URL which looks somehow like this: `yourappname.eu1.frbit.net`. This is where you can always reach your App thru the web browser. Use the App URL for development, testing and to connect to external services.

## Route a custom domain

Many different and advanced configurations are possible, see below. Here is the dead simple setup:

#### External provider

1. Point `www.yourproject.com` as a `CNAME` to your fortrabbit App URL `yourproject.eu1.frbit.net`;
2. forward your naked domain `yourdomain.com` to the actual subdomain `www.yourproject.com`.

#### fortrabbit dashboard

1. Enter the CNAME source `www.yourproject.com` as an external domain in the domain settings of the App.

PATIENCE YOUNG JEDI: Most Domain DNS records have a TTL (Time To Live) of 24 hours. This means that all name servers will only look every 24 hours if a domain target has been changed. In some cases it might even take longer.

## Set a custom root path

Normally all domains of the App will route to the `~/htdocs` [folder](directory-structure). Most of the time, this is what you want. However, in some cases, you need to set a different docroot or want to route different domains to different folder. Laravel for examples has a `public` folder for this.

You can do this by writing the relative path to the sub-folder (all folders below the htdocs folder are allowed) in the Root Path field right to the domain URL.

For example, if you want to use the folder `~/htdocs/web`, just enter web in the input. If you want to use the folder `~/htdocs/app/webbroot`, just enter `app/webroot`. 

## Change the default domain

This is an optional setting. Per default your App URL is the default domain. In the dashboard you can change this — so links and thumbnail preview generation works correctly.

## Advanced routing alternatives

The world of DNS is one of it's own. Let's dive into it so you can understand the backgrounds and explore alternative settings.

### Non naked domain

`www.fortrabbit.com` — Although this looks a bit antiquated, it's the modern "cloud enabled" way to go.

Back in the days the www. prefix indicated that's the address to type in the browser. Nowadays the www. prefix indicates i am ready to move this App in seconds to another server location. Test it: all big players run on a www. subdomain. The name of the subdomain prefix is not so important, but `www` is the convention for (marketing) entry points.

Non naked domains are usually routed thru CNAME records.

### Naked domain

`fortrabbit.com` — Indeed this looks visually more appealing, cleaner, leaner than his ugly brother the non-naked domain. Unfortunately this most likely means that the domain is routed via A-Record, which means an IP address is in place. In the world of cloud computing this is not smart, you (in this case we do it for you) want to be able to move your App around quickly, to scale up and down for instance, or in case of an emergency (dDos attack for example).

#### Problem: CNAME vs naked domains

It is possible to create a CNAME record for a naked domain. The problem is that `CNAME` records do not behave like the other records (`A`, `MX`, `TXT`, …). They are *greedy*. This means: They *overwrite* all other records.

An example: Assume you have a domain `domain.tld` and want to receive mails for it. So you create an `MX` record. Say it points to `mail.domain.tld`. If you now would create a `CNAME` record directly for `domain.tld` pointing to `otherdomain.tld`, every lookup of any record for `domain.tld` will be made on `otherdomain.tld` instead (aside from the `CNAME` record itself).

#### Solution 1: Use ANAME/ALIAS provider

There are DNS providers, which allow you to use non-standard records, which they call `ANAME` or `ALIAS`. Those combine the *positive* attributes of `CNAME`  with `A` records. In short: You can route a naked domain to a hostname without loosing your `MX` records.

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

#### Solution 4: Use your App's IP

**Not recommended**, since we cannot guarantee that your App's IP will stay the same forever (but we try hard).

```
# Find out IP on Mac or Linux:
host my-app.eu1.frbit.net
# my-app.eu1.frbit.net has address 54.228.198.23
```

### Wildcards

You probably want to route all requests for subdomains to your fortrabbit App like so: `*.mydomain.com` — for instance when the users of your App create spaces within in your domain name. That's it is possible, but for security reasons we'll need to verify your request. Also after you have a setup a wildcard for a domain, all other new requests for custom routings for this domain also need to be verified.

### HTTPS

For SSL connections on your own custom domain you'll additionally need: certificate, signed form an [external vendor](external-services#toc-ssl) and the [TLS component](tls).