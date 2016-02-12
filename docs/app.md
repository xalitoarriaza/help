---

template:      article
naviTitle:     About Apps
title:         What is an App?
lead:          Forget servers. Think servers instead. Learn the basic fortrabbit concepts.

seeAlsoLinks:
    - dashboard
    - migrating
    - scaling
    - app-design
    - old-apps
    - new-apps
    - deployment
    - terminology

tags: 
    - beginner

---



```nohighlight
# simplified App topology

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃App                                                 ┃
┃  ┏━━━━━━━━━━━━━━━┓ ┏━━━━━━━━━━━━━┓ ┏━━━━━━━━━━━━━┓ ┃
┃  ┃               ┃ ┃Webserver    ┃ ┃Database     ┃ ┃
┃  ┃               ┣─┫& PHP        ┣─┫cluster      ┃ ┃
┃  ┃               ┃ ┃             ┃ ┃             ┃ ┃
┃  ┃Load Balancer  ┃ ┗━━━━━━┳━━━━━━┛ ┗━━━━━━┳━━━━━━┛ ┃
┃  ┃               ┃ ┏━━━━━━┻━━━━━━┓ ┏━━━━━━┻━━━━━━┓ ┃
┃  ┃               ┃ ┃Webserver    ┃ ┃Database     ┃ ┃
┃  ┃               ┣─┫& PHP        ┣─┫cluster      ┃ ┃
┃  ┃               ┃ ┃             ┃ ┃             ┃ ┃
┃  ┗━━━━━━━━━━━━━━━┛ ┗━━━━━━━━━━━━━┛ ┗━━━━━━━━━━━━━┛ ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

An App is a virtual container for your web project, website, web application, staging branch, project or whatever you do. It consists of multiple Components like the PHP runtime or a MySQL database. 

Think of Components as micro services. Some Components are integral — meaning: you can't turn them off — while others are optional. Most Components are available in multiple expansion states (scaling plans). There are presets with common combination of Components.

## What's included

Each App comes with a set of basic features and limits: 

* a dedicated Git repo
* a unique [App URL](domains#toc-app-url) 
* custom metrics
* included monthly traffic
* various settings in the Dashboard
* [collaboration](collaboration) options


## Old and New Apps

You can currently run two different generations of Apps on fortrabbit: classical [Old Apps](old-apps) and [New Apps](new-apps). Old Apps were introduced 2013 and will end at around mid 2016, until then all Old Apps have to be migrated to the new ones. New Apps have been introduced in August 2015. 

## Multi-tenancy

Of course you can have multiple App under one Account. Even more, you can share Apps among [a team of a Company](collaboration).

## Further readings

* [Official pricing page](http://www.fortrabbit.com/pricing) - overview about scaling plans & options
* [Universal specs page](http://www.fortrabbit.com/specs) - detailed informations
* [When and how to scale](/scaling) - help article about scaling
* [fortrabbit VS Digital Ocean](http://www.fortrabbit.com/why-not-digitalocean) - marketing article comparing fortrabbit with VPS hosting

