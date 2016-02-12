---

template:       article
naviTitle:      External services
title:          Combine fortrabbit with other services
lead:           Learn how to integrate third party services to enhance your Apps and make your live easier.

tags:
    - beginner

keywords:
    - addon
    - add-on
    - integrations
    - integration
    - API
    - CI
    - cloud

---

As a real developer you want to solve each problem from scratch on your own. But there are some great Software as a Services offerings for our needs out there. Why reinvent the wheel then? There is an App for that.

There is not a common yes or no about using a cloud service. Often it's a trade off between the price, how good the solution fits your needs and on the other hand how much time you would have to invest to build something on your own. You might consider security and privacy concerns.

<!-- TODO: rewrite on US launch  -->

For latency-relevant services (databases, caches), we recommend to use a service hosted in Europe, even better in the Europe region of AWS. We don't have any business relations or interests going on here and we don't have expereinces with all of the services listed. fortrabbit is PHP as a Service, there are certain things we don't do by design, so for some tasks that are part of classical hosting you might need an external service now.

Things are changing quickly in this space, categories blend, some providers invent their own or have offerings in multiple categories.

## Application helpers

Sometimes there is an App for that: key-value stores, site search, Redis, Apache Solr, ElasticSearch, Image processing …

* [Openredis](https://openredis.com)
* [Redis-cloud](http://redis-cloud.com)
* [Indexdepot](https://www.indexdepot.com) Search solutions
* [imgix](https://www.imgix.com/) Image processing
* [Blitline](https://blitline.com) Image processing


## Databases

fortrabbit offers [MySQL](mysql) out of the box. But there are other interesting database types out there. With database abstraction as part of your framework you actually can switch quickly to an alternative. NoSQL databases are quite popular. Commonly used DBaas:

* [ElephantSQL](http://www.elephantsql.com) PostgreSQL Databases in AWS Europe
* [Cloudant](https://cloudant.com) CouchDB 
* [MongoLab](https://mongolab.com) MongoDB
* [Compose.io](https://www.compose.io) MongoDB, Elasticsearch & RethinkDB


## Transactional mails

Ok, you want send your "forgotten passsword link mail" from your App without fuzz. Transactional mails are automated customer relationship messages send by your App. In contrast to newsletters transactional mails are triggered by user action. It's where marketing meets communications. 

With fortrabbit you can't use `sendmail` out of the box, so you either use SMTP in combination with a classical mail provider or a specialized provider. They usually offer an API or simply [SMTP](quirks#toc-mailing):

* [Postmark](https://postmarkapp.com) 
* [Sendgrid](http://sendgrid.com)
* [Mailjet](https://www.mailjet.com)
* [Mandrill](https://www.mandrill.com)
* [Mailgun](http://www.mailgun.com)

## Cloud storage

Ok, this not about iCloud, DropBox or Google Drive here. You don't want to put your large binaries (like for download files) in Git? Good idea! You want to put your generated and compressed static assets like images, javascripts and stylesheets somewhere else? Good idea!

* [Cloud Files](http://www.rackspace.com/cloud/files)
* [DreamObjects](https://www.dreamhost.com//cloud/dreamobjects)
* [Google Cloud Storage](https://developers.google.com/storage)
* [Amazon S3](http://aws.amazon.com/s3) - check out our BLOG[guide to setup S3 as a cloud storage](new-app-cloud-storage-s3)


## Message queuing

Ok, you want to make real good use of your [Worker Node](workers) with fortrabbit? Combine it with a queue service:

* [Iron MQ](http://www.iron.io/mq)
* [CloudAMPQ](http://www.cloudamqp.com)
* [Amazon SQS](http://aws.amazon.com/sqs)

## Uptime monitoring

Ok, you want to know if fortrabbit is really up and delivering as promised in the Service Level Agreement? Go ahead, ping your App constantly: 

* [PingDom](https://www.pingdom.com)
* [StatusCake](https://www.statuscake.com)
* [UptimeRobot](http://uptimerobot.com)
* [Mist](https://mist.io)

## Performance monitoring

Ok, you care about App performance, mostly of your server side scripts? You need to know what is slowing and where your bottle necks are? The first thing is to get some data in.

We have a special [New Relic integration](new-relic).

## Code quality profiling

Apart from open source solutions like Xdebug or Zend Debugger there are commercial services to help you with debugging and finding performance bottlenecks in your code:

* [Blackfire](https://blackfire.io/) ‹ by sensiolabs, [blackfire + fortrabbit](blackfire)
* [Qafoo](http://qafoo.com/)


## Code collaboration

Ok, each fortrabbit App has [Git](git) built in. Git itself is a very powerful tool to collaborate on code with others. GitHub is the poster-child Git platform — enhanced Git hosting, making repos explorable and accessible (pull requests, issues, stats, activity log …).  With Git standards alone you can combine fortrabbit and nearly any other code collaboration tool:

* [GitHub](https://github.com)
* [Bitbucket](https://bitbucket.org)
* [Plan.io](http://plan.io) — hosted Redmine (ticket system) with Git integration

Please also see the [dedicated article about GitHub/Bitbucket integration](bitbucket-github-and-fortrabbit).

## Logging

* [App Enlight](https://appenlight.com)
* [Loggly](http://loggly.com)
* [Sumo Logic](http://www.sumologic.com)
* [Papertrail](https://papertrailapp.com)
* [AirBrake](https://airbrake.io/pages/home)

## Continuous Integration

Ok, Git push is all fine. But sometimes you might want to run a test or two before everything get's deployed into production. We care about [Continuous Integration](continuous-integration). You might try one of those CI services:

* [Codeship](https://www.codeship.io)
* [Codeclimate](https://codeclimate.com)
* [Wercker](http://wercker.com)
* [CircleCi](https://circleci.com)
* [Travis CI](https://travis-ci.org)

## Crons

Ok, you want weekly status reports, daily cache cleanups or monthly invoice generations? Cron tasks, or short crons, come in handy if you want to execute a task on a specific, reoccurring time. 

fortrabbit offers a great [worker & cron](workers) solution, but sometimes this solution might be a bit over-sized to execute a small cron script from time to time. You might use an external service to time your execution of scripts hosted on fortrabbit. Go ahead, but be warned: Cronjob as a Service providers look a mostly antiquated. The good news is that there are free offerings:

* [Easycron](https://www.easycron.com)
* [Mywebcron](http://www.mywebcron.com)
* [Setcronjob](http://www.setcronjob.com)
* [Webcron](http://www.webcron.org)

## SSL certificates

Ok, you want to utilize transfer encryption for your custom domain. On the one hand you'll need the fortrabbit [TLS component](tls), on the other hand you need a service willing to sign your certificate, such as:

* [Digicert](https://www.digicert.com)
* [SSLstore](https://www.thesslstore.com)
* [SSL.com](https://www.ssl.com) — US
* [VeriSign](http://www.verisign.com)
* [Comodo](https://ssl.comodo.com)
* [Thawte](http://www.thawte.com/products)
* [GeoTrust](http://www.geotrust.com)
* [Let's encrypt](https://letsencrypt.org/) < a free alternative

AGREE: All those commercial vendors look don't look trustfully. Well, that seems to be part of the shady business they are doing.


## Domain registration

Ok, you want to register your domain somewhere.

* [iwantmyname](https://iwantmyname.com/)
* [Hover](https://www.hover.com/)
* [Name.com](https://www.name.com/)
* [namecheap](https://www.namecheap.com/)


## DNS as a Service

Ok, you have your own domains registered somewhere already. But that service doesn't support CNAME entries or forwards like ALIAS or ANAME? You can use a Domain Name System service to do this:

* [DNS Made Easy](http://www.dnsmadeeasy.com)
* [DNSimple](https://dnsimple.com) — also offering SSL



## CDN

Ok, you want your static assets to be near your clients for better performance? Try a Content Delivery Network:

* [Cloudflare](https://www.cloudflare.com) — also offering SSL now
* [MaxCDN](https://www.maxcdn.com)
* [Fastly](https://www.fastly.com/)
* [KeyCDN](https://www.keycdn.com/)

## Payment processing

Ok, you are collecting payments form your users, possibly via credit card. Of course you will not store any data in your fortrabbit App. There is payment service providers:

* [Stripe](https://stripe.com)
* [WireCard](http://www.wirecard.com/)



## Other sources

Ok, don't take only our word. Load testing, browser testing, video processing, payment processing, metrics, code collaboration, deployment helpers there is much more out there. Have a look around:

* [Developer facing services](https://docs.google.com/spreadsheet/ccc?key=0An6rx68cKNFNdDNYdFdSSTNzZXl5eGRSY0ZxSW10aHc) — an open google spreadsheet
* [Leanstack.io](http://leanstack.io) — developer tools database & platform
* [The idea of decoupled hosting](http://blog.fortrabbit.com/the-idea-of-decoupled-hosting)
