---

template:       article
naviTitle:      SSH keys setup
title:          Troubleshooting SSH keys
lead:           We get a lot regular support requests regarding SSH key setup. This article helps solving common issues and is a resource where to get more help.


keywords:
    - ssh key
    - public key
    - git
    - ssh

seeAlsoLinks:
    - ssh-sftp-old-app
    - deployment
    - collaboration
    - security
    - app
    - ssh-git

tags:
    - beginner
    - git

---

On fortrabbit SSH keys are used to authenticate with a variety of services such as [deploying via Git](git), [accessing live logs](logging) and [remote MySQL access](mysql#toc-remote-mysql-access). To utilize those services you first need to install your SSH (public) key in the Dashboard.

## Shortcut: GitHub import

Have you already setup your SSH keys with Github? If so, then you can easily [import your SSH keys from within the Dashboard](http://dashboard.fortrabbit.com/boarding/keys/github).

## Generating SSH keys

The procedure to create SSH keys is a bit different on each Operating System. These articles help you creating your SSH key pair:

* [Git SCM help](http://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key)
* [GitHub help](https://help.github.com/articles/generating-ssh-keys/)
* [Atlassian help](https://confluence.atlassian.com/display/STASH/Creating+SSH+keys)

Please mind to create those keys on your local machine, not remotely!

## Saving your public SSH key with your Account

After you have created your local SSH key, you'll need to install it on fortrabbit. MIND THE DIFFERNCE BETWEEN PUBLIC AND PRIVATE KEY! You can install the public SSH key in the Dashboard under your Account. This is what a valid SSH key looks like:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDbez9IDLYECMpQUQgNTWPG5aPMwJFNNP3a0
gAHVz8+N4HgiwFwBll2iUX0YPHIpbfeXN4Kab30qsevw59cjQ1XC7yjkrXy03OyOv/Z9X+KpB
vnf/cRXwz2zxfQqwvmXIQl3jlxyuA+Y4VjvELIvCrnnsfJDETmF8HZG4zA5XFfS95y5xx3TF9
S/eTlx2qWrmhsf20H+P/FK8otXKa+EW4UY6mew/lVxboEYDfCTju8cS5raJBmTehBaYyWI2dy
9oEWvD+qySvrEf1gXRRAMmt0/bOR4jw8G18i5siMtse2s/qMomG08VMeVIAEK9Tp64Mx4mmQv
IvP1bffus+WdY75 you@localhost
```

The above is multi-line only for readability. Please don't use multi-line SSH keys in the Dashboard.

## Some more about SSH key authentication

In case you haven't worked with SSH keys before — you'll first might be interested to understand how it works. The bottom line is that SSH key authentication is a bit nerdy, but actually both: convenient and secure. "SSH keys are a way to identify trusted computers, without involving passwords." That's from GtiHub and probably the shortest way to explain what it is about and the most crucial benefit.

In public key authentication you have a key pair that consists of a public (id_rsa.pub) and a private key (id_rsa). What is encrypted with one (eg the public key) can be decrypted by the other (then: the private key). Further, having only the public key [does not allow you to derive the private key](https://en.wikipedia.org/wiki/List_of_unsolved_problems_in_mathematics). Hence you can safely "give out" your public key.

When you install your public key with fortrabbit it can be used to authenticate you: your SSH clients uses your private key to encrypt plain text data, which is then decrypted, using your public key, on the fortrabbit SSH server. If this decryption succeeds, then it must have been encrypted by your private key and you are let in.



## Account SSH keys

**This is the recommended method:** you store and manage your public SSH keys with your user Account on fortrabbit. This way you always have up-to-date code access on each App you own or you are collaborating with. It also makes managing collaboration easy — add/remove collaborators and code access is handled "automagically".


## App-only SSH keys

In certain cases you might want to add code access to an App without the need to register a new Account with fortrabbit. One case is some hectic ad-hoc hotfix scenario (good luck!), another case is that you have some advanced deployment with a third party continuous integration service bot going on. So you can install additional App-only custom public SSH keys with each App. You manage those App-only SSH keys in the Dashboard.
