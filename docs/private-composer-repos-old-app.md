---

template:    article
title:       How to use private Composer repos with SSH keygen
naviTitle:   Private Composer repos
lead:        Generate a unique SSH key-pair for your App on fortrabbit to use private Composer repos.
dontList:    true
oldApp:      true
linkOther:   /private-composer-repos

tags:
    - git

seeAlsoLinks:
    - deployment
    - composer

keywords:
    - Composer
    - Git
    - ssh
    - GitHub
    - Bitbucket
    - Hooks

externalLinks:
    - http://getcomposer.org/doc/05-repositories.md#using-private-repositories
    - http://getcomposer.org/doc/05-repositories.md#hosting-your-own

---

Modern PHP App development utilizes [Composer](composer) as a dependency manager. There are many great open source packages [out there](http://packagist.org). But your company code is probably not intended to be released to public. That's when you use private Composer repositories — protected by SSH public key authentication. To use your private Composer repo you need to set up authentication so your fortrabbit App can access your external repo (probably hosted on Bitbucket, GitHub etc). 


### Generating a public key for an Old App

To generate a SSH key-pair for an [Old App](new-apps), send a `sshkeygen` trigger in a commit message. This can either be a commit containing changes, or an empty one, just for the key generation. Here you go:

```bash
git commit --allow-empty -m 'Generate SSH Key [trigger:sshkeygen]'

git push
# Counting objects: 1, done.
# Writing objects: 100% (1/1), 210 bytes, done.
# Total 1 (delta 0), reused 0 (delta 0)
# remote: Step1: Updating repository
# remote:  ->; OK
# remote: Step2: Deploying
# remote:  ->; OK
# remote: Step3: Composer Hook
# remote:  ->; Skip, not triggered.
# remote: Step4: SSH Key-Gen Hook
# remote:  ->; Generating new SSH key pair
# remote: ########## Public Key ############
# remote: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCoRG3kKXftyfp9XGTqc8aOVPqJe+UXmNExE651uILGOE3YnKgvRF9jIeBhdw+63MFcNaYcwvIaADCdQdIXykgHymi2K/BLFr+92W2W3UriBjGOsy9rixHlQK3OFY7OeitmMATipAHYm6dNyklhaUQ/B8XZe3kXkdlC6tpIS8eUy1GD+OggtkAXTH9kqeecAdpUpLQg8DgMmjOxgwcGiCU2a5WVVwelIirj419zEVtDh1NUA9T75tp8r5wYHBf6YZzD5SLO/j+3fWPWVMGOZTtsyZOwZx9aJs54c2wn5BO5rDMFHR0RNHBpq3Jbqae8W3Tqzs8LWQRLilCQTlh3We8p my-app
# remote: ########## Public Key ############
# remote:  ->; OK (Save the key now! Can only be re-generated!)
# remote: >; All Done <;
# To git@git1.eu1.frbit.com:my-app.git
#   69afcf2..7d0ad38  master ->; master
```

The private key will not be displayed (you don't need it either). The public key in the example is starting at `ssh-rsa` and ending at `my-app`:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCoRG3kKXftyfp9XGTqc8aOVPqJe+UXmNExE651uILGOE3YnKgvRF9jIeBhdw+63MFcNaYcwvIaADCdQdIXykgHymi2K/BLFr+92W2W3UriBjGOsy9rixHlQK3OFY7OeitmMATipAHYm6dNyklhaUQ/B8XZe3kXkdlC6tpIS8eUy1GD+OggtkAXTH9kqeecAdpUpLQg8DgMmjOxgwcGiCU2a5WVVwelIirj419zEVtDh1NUA9T75tp8r5wYHBf6YZzD5SLO/j+3fWPWVMGOZTtsyZOwZx9aJs54c2wn5BO5rDMFHR0RNHBpq3Jbqae8W3Tqzs8LWQRLilCQTlh3We8p my-app
```

You can now install the key in your (companies) private Composer repository. You can re-run this hook at any time. However, it will always generate a *new* key-pair and remove the old one.


### Generating a public key for a New App

With the New Apps you can create a new SSH keypair for your App, using the `keygen` command.

```bash
ssh git@deploy.eu2.frbit.com keygen your-app-name
```

This will print out a public key, which you can then install with your private repository (on Github, Bitbucket or wherever).


### Link your private repo

Now you can add your private repositories into your `composer.json` file like usual:

```
    "repositories": [
        {
            "type": "vcs",
            "url":  "git@github.com:my-company/my-package.git"
        }
    ],
    "require": {
        "my-company/my-package": "1.2.*"
    }
}
```
