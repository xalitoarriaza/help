---

template:      article
naviTitle:     Multi-staging
title:         Multi stage App life cycles
lead:          Learn about development/production environments and how to run them on fortrabbit.
dontList:      true
oldApp:        true
linkOther:     /multi-staging
tags:
     - advanced


blogLinks:
    - multi-stage-deployment-for-website-development


---



## Goal

**Move fast, break nothing.** Experimenting without downtimes. Multi staging is quite the opposite of open heart surgery — it's an advanced strategy to run the same application in separate similar environments. It's especially useful in teams but can also bring benefits for the one-man-army.

### Use cases

* **Feature development**: Build a new version while still being able to fix bugs in the running App without uploading all the new feature code.
* **Purpose separation**: The backend team can break things while the front team can still work on.
* **Continuous integration**: Code needs to be tested and monitored before being deployed to the live App.


### Common set ups

* **staging + production**: "Staging" is where you do your development. "Production" runs the live App. From time to time you migrate all the (stable) changes from staging into production.
* **temporary + production**: Same as above, it's more of a one-time-developed project. Maybe once a year there is an upgrade, a re-brush or alike. For this you utilize an additional, temporary environment, in which you do the upgrades.
* **testing + staging + production**: Code changes are made to the testing environment. Once they seem stable, they get published to the staging where they are monitored and further tested. Finally they get published to production.

### Differences to local development

Most likely you are developing with a local PHP environment on your machine. So your laptop is already a kind of **staging** while the actual App at fortrabbit is **production**.

A key aspect of multi-staging is that environments are nearly identical environments of production and development. A Mac with Mamp for example is for sure a bit different to the fortrabbit environment.

Virtualization is a way to make your local environment behave more like the one in production. [Vagrant](https://www.vagrantup.com/) is an open source tool to "create and configure lightweight, reproducible, and portable development environments".


## Multi-staging on fortrabbit

People often assume that various Git branches are all it takes to set up different versions of an App on fortrabbit. Unfortunatly that's only half of the way.

Multi staging doesn't necessarily require Git – but it's a good match. Using local branches to map the different App versions is a good idea, as they all share the same code base.

On fortrabbit only the [master branch get's deployed](git#toc-only-the-master-branch-will-be-deployed) on webspace. Use multiple Apps, a dedicated one for each branch, map local branches to remote master branches.


```nohighlight
+------------------+  +------------------+
| App1.master      |  | App2.master      |
+---------+--------+  +---------+--------+
          |                     |
+---------+--------+  +---------+--------+
|  local.staging   |  | local.production |
+------------------+  +------------------+
```

### Sample setup

You have three local branches: **test**, **stage** and **prod**. Each local branch maps to the **master** branch of a dedicated App. You commit your changes only to the **test** branch and merge them upwards. You start out cloning the first App and add the others as different remotes to your local Git. The last step is to create and associate local branches with the remote branches.


#### Clone the first App

Assuming you have created three Apps. In this example, let's name them **my-app-test**, **my-app-stage** and **my-app-prod**.


```bash
cd ~/Projects
git clone git@git1.eu1.frbit.com:my-app-test.git my-app
cd my-app
```

The first cloning will create the remote `origin` and clone the `master` branch to you local disk.

#### Setup all remotes

Now let's rename the cloned `origin` remote to `testing` and add the two additional `staging` and `production` remotes:

```bash
git remote rename origin testing
git remote add staging git@git1.eu1.frbit.com:my-app-stage.git
git remote add production git@git1.eu1.frbit.com:my-app-prod.git
```


#### Mapping branches for New Apps

For [New Apps](new-apps) you can name your local branch like your App name and push it to remote. Your fortrabbit will prefer a branch named like the App over the master branch.

#### Setup and map all the branches for Old Apps

WORD: If you cloned from a completely virgin App, you need to make a first, local commit. This will create and settle the local branch `master`.

First rename the `master` branch to `test`. It will keep its tracking to the `testing` remote's `master` branch, which is intended. Then create the two additional (`stage` and `prod`) branches and map them to the appropriate remote's `master` branch.

```bash
# setup test -> testing
git branch -m master test
git config remote.testing.push refs/heads/test:refs/heads/master

# setup stage -> staging
git checkout -b stage
git config branch.stage.remote staging
git config branch.stage.merge refs/heads/master
git config remote.staging.push refs/heads/stage:refs/heads/master

# setup prod -> production
git checkout -b prod
git config branch.prod.remote production
git config branch.prod.merge refs/heads/master
git config remote.production.push refs/heads/prod:refs/heads/master</pre>
```

To verify everything is setup properly, check your `.git/config` file. It should look similar to this:

```
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "testing"]
    url = git@git1.eu1.frbit.com:my-app-test.git
    fetch = +refs/heads/*:refs/remotes/testing/*
    push = refs/heads/test:refs/heads/master
[branch "test"]
    remote = testing
    merge = refs/heads/master
[remote "staging"]
    url = git@git1.eu1.frbit.com:my-app-stage.git
    fetch = +refs/heads/*:refs/remotes/staging/*
    push = refs/heads/stage:refs/heads/master
[remote "production"]
    url = git@git1.eu1.frbit.com:my-app-prod.git
    fetch = +refs/heads/*:refs/remotes/production/*
    push = refs/heads/prod:refs/heads/master
[branch "stage"]
    remote = staging
    merge = refs/heads/master
[branch "prod"]
    remote = production
    merge = refs/heads/master
```

#### Commit to testing

Let's play thru a commit cycle. Code away, build something great.

```bash
git checkout test
# make changes..
git commit -am 'My changeset'
git push
```

#### Merge upwards to staging

When everything works out as planned and your codes reaches a sufficient level of majority.

```bash
git checkout stage
git merge test
git push
```

#### Merge upwards to production

Now everything should have been thoroughly tested and is ready for production.

```bash
git checkout prod
git merge stage
git push
```

## Caveat runtime data

The biggest problem introduced with multi stage environments is runtime data: databases, user uploads and anything else that is being generated by the App. Sure you need to separate those as well. If all environments require the latest data to run properly or for testing, you need to figure out a way to synchronize the production data downwards, to the other environments. These problems are highly individual and cannot be solved in a general manner. We are experienced with these kind of things and [we love to help you](http://www.fortrabbit.com/contact).