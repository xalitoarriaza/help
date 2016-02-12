---

template:      article
naviTitle:     Multi-staging
title:         Multi stage App life cycles
lead:          Learn about development/production environments and how to run them on fortrabbit.
linkOther:     /multi-staging-old-app
tags:
     - advanced


blogLinks:
    - multi-stage-deployment-for-website-development


---


## Goal

**Move fast, break nothing.** Experimenting without downtimes. Multi staging is is quite the opposite of open heart surgery — it's an advanced strategy to run the same application in separate but similar environments. It's especially useful in teams but can also bring benefits for the one-man-army.

### Use cases

* **Feature development**: Build a new version while still being able to fix bugs in the running App without uploading all the new feature code.
* **Purpose separation**: The backend team can break things while the frontend team can still work uninterrupted.
* **Continuous integration**: Code needs to be tested and monitored before deployed into to the live App.


### Common set ups

* **staging + production**: "Staging" is where you do your development. "Production" runs the live App. From time to time you migrate all the (stable) changes from staging into production.
* **temporary + production**: Same as above,  it's more of a one-time-developed project. Maybe once a year their is an upgrade, a re-brush or alike. For this you utilize an additional, temporary environment, in which you do the upgrades.
* **testing + staging + production**: Code changes are made to the testing environment. Once they seem stable, they get published to the staging where they are monitored and further tested. Finally they get published to production.

### Differences to local development

Most likely you are developing with a local PHP environment on your machine. So your laptop is already a of kind of **staging** while the actual App at fortrabbit is **production**.

A key aspect of multi-staging is that environments are nearly identical environments of production and development. A Mac with MAMP for example is for sure a bit different to the fortrabbit environment.

Virtualization is a way to make your local environment behave more like the one in production. [Vagrant](https://www.vagrantup.com/) is an open source tool to "create and configure lightweight, reproducible, and portable development environments".


## Multi-staging on fortrabbit

The short of it: All you need to utilize multi-staging on fortrabbit is multiple Apps.

Git supports multiple, named branches of your code. Per default, it comes with a branch called `master`. When pushing to fortrabbit our deployment will look for the `master` branch and deploy it. To make things easier for multi-staging scenarios, there is another branch, which is preferred over the `master` branch by the fortrabbit deployment: A branch named like your App. Say the name of your App is `my-app`, then you can create a branch called `my-app` which will be deployed instead of the `master` branch.

### Sample setup

Assuming you use a 3-stage layout: testing, staging and production. You start by creating three Apps. Let's name them: `my-app-test`, `my-app-stage` and `my-app-prod`.

Now you can map those App names with local branch names. Here is how:

#### Clone the first App

Let's start by cloning the testing App into a local folder named `my-app`:

```bash
cd ~/Projects
git clone git@deploy.eu2.frbit.com:my-app-test.git my-app
cd my-app
```

This first cloning will create the remote `origin` and clone the `master` branch to you local disk.

#### Add the other Apps

First you should rename the cloned `origin` remote to `testing`. Then you can add the two additional staging and production Apps:

```bash
git remote rename origin testing
git remote add staging git@deploy.eu2.frbit.com:my-app-stage.git
git remote add production git@deploy.eu2.frbit.com:my-app-prod.git
```

#### Mapping branches for New Apps

Again, start out with the testing App by creating a new local branch named as the testing App. Then you can push it to the remote using the `-u` flag, which will create a "link" between the local and the remote branch:

```bash
git checkout -b my-app-test
git push -u testing my-app-test
```

Now repeat this with the two other Apps:

```bash
git checkout -b my-app-stage
git push -u stating my-app-stage
git checkout -b my-app-prod
git push -u stating my-app-prod
```

To verify everything is setup properly, check your `.git/config` file. It should look similar to this:

```
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "testing"]
    url = git@deploy.eu2.frbit.com:my-app-test.git
    fetch = +refs/heads/*:refs/remotes/testing/*
[remote "staging"]
    url = git@deploy.eu2.frbit.com:my-app-stage.git
    fetch = +refs/heads/*:refs/remotes/staging/*
[remote "production"]
    url = git@deploy.eu2.frbit.com:my-app-prod.git
    fetch = +refs/heads/*:refs/remotes/production/*
[branch "my-app-test"]
    remote = testing
    merge = refs/heads/my-app-test
[branch "my-app-stage"]
    remote = staging
    merge = refs/heads/my-app-stage
[branch "my-app-prod"]
    remote = production
    merge = refs/heads/my-app-prod
```

#### Commit to testing

Let's play thru a commit cycle. Code away, build something great.

```bash
# go to testing branch
git checkout my-app-stage
# make changes..
git commit -am 'My changeset'
# the push will automatically push to the testing App, since the branch was linked to it
git push
```

#### Merge upwards to staging

When everything works out as planned and your codes reaches a sufficient level of majority you can merge your testing code into staging:

```bash
# go to staging branch
git checkout my-app-stage
# merge changes from testing
git merge my-app-test
# the push will automatically push to the staging App, since the branch was linked to it
git push
```

#### Merge upwards to production

Now everything should have been thoroughly tested and is ready for production.

```bash
# go to production branch
git checkout my-app-prod
# merge changes from staging
git merge my-app-stage
# the push will automatically push to the production App, since the branch was linked to it
git push
```

## Caveat runtime data

The biggest problem introduced with multi stage environments is runtime data: databases, user uploads and anything else that is being generated by the App. Sure you need to separate those as well. If all environments require the latest data to run properly or for testing, you need to figure out a way to synchronize the production data downwards, to the other environments. These problems are highly individual and cannot be solved in a general manner. We are experienced with these kind of things and [we love to help you](http://www.fortrabbit.com/contact).