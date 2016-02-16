---

template:   article
title:      How to move your App to fortrabbit
naviTitle:  Migration
lead:       How to transfer an application to fortrabbit.
linkOther:  /migrating-old-app
tags:
     - beginner

seeAlsoLinks:
     - app
     - scaling
     - app-design
     - deployment
     - terminology

---


WORD: fortrabbit is not your traditional hosting environment. It may require some tinkering on your application and we recommend reading about [the fortrabbit platform generals](app) as well as our getting started guides.

This article covers the general basics as well as some deep links for moving your App from any hosting provider to fortrabbit. This will hopefully cover everything you need to realize a smooth transition.

Each App is different, adjust your plan accordingly and don't hesitate to [contact us](http://www.fortrabbit.com/contact) with your specific questions.

## Create your fortrabbit App

Each website or web application is represented by an [App](app) on fortrabbit. You can have as many Apps as you want. So in order to move your project you need to create an App on fortrabbit first.

## Prepare your domains

To minimize downtimes caused by DNS cache propagation you should change the Time to Live (TTL) of your domains to the lowest possible value. Something like five minutes would be just great.

If you have some kind of web control panel with your domain provider, then you can most likely change it yourself. If not, just get in touch with the provider and ask them to reduce your TTL for you. You should do this 48-72 hours before you start with the migration.

After that's done make sure to add all the domains to your new fortrabbit App in the dashboard.

## Migrate your runtime data

Runtime data means all kinds of data, which is created by your App at runtime. Usually these are user uploads or uplodas from a CMS or somesuch.

<!-- TODO: change on asset storage launch -->

Apps on fortrabbit do not have a persistent storage, so you need an alternative. We highly recommend to use a cloud storage, such as Amazon's S3. The setup is [neither hard nor does it take long](http://blog.fortrabbit.com/new-app-cloud-storage-s3).

## Migrate your code

If you already subscribe to a Git based workflow, then there is probably not much to do here. If not, then you should familiarize yourself with Git. We promise: You won't regret it. Once you've used it, you won't go back without being forced.

## Migrate your databases

If your App is using a MySQL database, you will need to migrate the database data as well. [Export the MySQL database from your old hosting and import](mysql#toc-export-amp-import) it to the fortrabbit database.

## Sending e-mails

If your App needs to send mails, you'll need a [3rd party provider](external-services#toc-transactional-mails).

## TLS/SSL (optional)

All Apps on fortrabbit can be accessed using a free HTTPS URL (`https://your-app-name.frb.io`). If you want to use a custom domain with a custom certificate we offer the [TLS Component](tls).

## Final switch: DNS

Now that you have migrated your code, runtime data and database - and all the other stuff you needed - you are ready to push the button.

Now that your App is fully mirrored on fortrabbit and ready to handle traffic, you can [route your Domains DNS records to fortrabbit](domains#toc-route-a-custom-domain). If you waited the 48-72 hours for DNS caches to clear, downtime will be minimal as traffic is routed to your App on fortrabbit.

## Migrating away from fortrabbit

We don't lock you in. You can cancel at any time. Due to the distributed nature of services it's even easier to move somewhere else then. The steps are essentially the same as above.