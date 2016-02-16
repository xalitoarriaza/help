---

template:    article
title:       Teamwork on fortrabbit
naviTitle:   Collaboration
lead:        Mapping real world team relationships in a hosting env.

tags:
    - beginner

seeAlsoLinks:
    - security
    - billing
    - teamwork-video

keywords:
    - collaboration
    - teamwork
    - sharing
    - permissions
    - roles
    - organization
    - access

---

> Code is better when shared.

This is definitely not the sexy feature which gets you hooked immediately. It's not even technical, it's more like philosophy. But it's what you won't miss and will love once you got to know it. See the fortrabbit collaboration features here.

## Problem

Most of our clients run multiple Apps on fortrabbit. But here is the thing: Life is complicated and professional life even more so. Say we have _App X_, which was once part of _Project A_ but is now not anymore. And actually you don't want to pay it, but your client _Acme Inc_ should. And not to mention: _Kevin Flynn_, the guy your client hired for code review, also should have code access.

Well, you just could set up a new Account for each App and be done with it. But the more Apps you have, the more of a mess you will end up with: no bird's eye view about what is going on, logout/login to switch to another App, manage all kind of stuff twice, ...

## Solution

Enterprise solutions for the rest of us! A [multi client model](https://medium.com/@frank_laemmer/our-multi-client-model-3b965d2f1060) that maps to your real world in nearly any context: freelancer, startup, digital agency. Here are the building blocks:

## User

When you sign up with fortrabbit you'll be asked for your first and your last name. You see, we expect that a user account represents a human being, not your company, not even when you are the one-man-army. Don't share your fortrabbit user account, it's your "private" one. Here you'll deposit YOUR public SSH keys, that's your very own access to the Dashboard.


## Company

Your business, no matter how small or large, is always represented as a Company within the fortrabbit platform. A Company has to have at least one Billing Contact (see below) and one User (see above) associated with. It can, of course, own multiple Apps.

Each User can be part or own multiple Companies on fortrabbit. That is useful for example when you use fortrabbit privately for your week-end projects and for business. Or if you as a freelancer collaborate with various digital agencies.

## User roles

It also works around the other way, not only you can take part in multiple Companies, each Company can have multiple Users associated with it.

»Everyone is equal, but some are more equal than others.« That's what permission based roles are for. That's the way we make sure only the boss can really mess up.

Every User must have a role associated within each Company hir is in:

### Owner

The Owner role can NOT be modified by someone else, multiple Owners per Company are possible:

* create, delete, configure & scale all Apps of the Company;
* code access to all Apps of the Company;
* delete, create, change Billing Contacts for the Company;
* manage all roles (but other Owners) of the Company;
* invite new Users for any role to the Company;
* leave the Company (if other owners are present).

### Admin

The Admin role can be modified by Owners, hir can:

* create, delete, configure & scale all Apps of the Company;
* code access to all Apps of the Company;
* invite new Admins or Developers to the Company;
* leave the Company.

### Developer

The Developer role can be modified by Owners and Admins, hir can:

* configure certain Apps of the Company;
* code access certain Apps of the Company.

### Billing Contact

Sometimes you have different billing preferences within one Company. That's what Billing Contacts are for. Billing Contacts are managed by the Company Owners, they consist of:

* A payment method
* A billing address
* A billing e-mail address

Each App (expect trials) has to be associated with a Billing Contact. Each Billing Contact has its own invoice archive.


## How to use it

Don't fear. It's not as complicated as it may sounds. All team features are optional, the [Dashboard](dahsboard) makes managing all this easy. See our [teamwork video](teamwork-video)!


