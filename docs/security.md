---

template:    article
title:       Some words on security
naviTitle:   Security
lead:        fortrabbit security concepts and how you can help.

keywords:
    - beginner
    - dashboard

seeAlsoLinks:
    - collaboration
    - app

tags:
    - beginner

---


»We take security very serious.« Isn't that what everybody is saying? It's our business to keep your business online. Our clients trust us with that. We know what we do and we are doing hosting for over ten years now. Make yourself a picture, have a look at our: [history](http://www.fortrabbit.com/about), [blog](http://blog.fortrabbit.com/), [tweets](https://twitter.com/fortrabbit), [past incidents](http://status.fortrabbit.com) or ask our customers directly. 

## Our responsibilities

### Physical infrastructure

fortrabbit utilizes Amazon Web Service (AWS) data centers. Amazon data centers have been accredited under several certificates (including ISO 27001). AWS stands for a high level of physical security to safeguard their data centers. Among others things they employ two-factor authentication for all their authorized staff members, military grade perimeter controls and security staff at all ingress points.

As for environmental protection AWS has sophisticated fire detection and suppression equipment, fully redundant power infrastructure with integrated UPS units and high end climate control system to guarantee an optimal working environment for the hardware.

For a more in detail view, we refer you to the [AWS Security Center](https://aws.amazon.com/security).

### SysOp level

fortrabbit employs a multi tier security strategy.

On the inside, each node is build around a hardened Linux kernel, which enforces strong privilege and resource separation mechanisms on OS level. All operating systems and software components are kept up-to-date and by our maintenance staff and we pride ourselves in reacting fast to all [Poodles](http://blog.fortrabbit.com/ssl-v3-disabled-poodle-vulnerability/), [Heartbleeds](http://blog.fortrabbit.com/heartbleed-openssl-vulnerability/), Shellshocks and [Ghosts](https://twitter.com/fortrabbit/status/560478509475577856) that have and will come up.

The next tier are isolated virtual containers, which guarantee complete logical separation of Apps on fortrabbit. In addition, the container technology allows for hard resource capping reducing the bad neighbor effect of shared environments to a bare minimum.

On the outside, we utilize network firewalling and hardened TCP/IP stacks to mitigate resource exhaustion attempts. Sniffing and spoofing attacks are prevented through the underlying infrastructure. Our setup is flexible and we are able to isolate or boost resources quickly.

### Credit card security

We use Wirecard — a PCI Level 1 compliant provider — for processing credit card payments.

## Your responsibilities

We are in it together! You are responsible for the code you write and even the one you are using. 

### Check your code

Make sure to follow [security guidelines](http://www.phptherightway.com/#security). It's a good practice to perform a security check against the most common attack vectors before going live. Also mind the [OWASP Cheat Sheets](https://www.owasp.org/index.php/OWASP_Cheat_Sheet_Series) to negate attacks before they can start.

### Frameworks & CMS systems

Update the frameworks and CMS systems frequently. You are to blame when your WordPress installation becomes out of date and gets hacked. Composer makes updating easy for modern frameworks.

## Dashboard access

The password you use to login with the [fortrabbit Dashboard](https://dashboard.fortrabbit.com) is your master password. We recommend to use a [pass-phrase](http://xkcd.com/936/) kind of password. Use something that is easy to remember for you, but hard to guess for anyone else while long enough to stand against brute force attacks.

### Session time

It's always a balance between usability and security. Per default we'll log you out after some time of inactivity. You can modify that timing in the Dashboard under your Account settings.

### Account password

Please choose a secure fortrabbit Account password. The best password is hard to crack but easy to remember. Pass-phrases can work well here.


### Re-enter Account password for critical actions

For "dangerous actions" in the Dashboard you need to re-authenticate with your Account password (and 2FA if enabled). As the fortrabbit [Dashboard](dashboard) is all about administrative tasks, this includes many tasks. So in short: we do [SUDO](http://en.wikipedia.org/wiki/Sudo). You can also modify how often you'll have to enter your SUDO password.

### Two-factor authentication

We highly recommend to enable 2FA with your fortrabbit Account. You can so do in the Dashboard — you'll be guided setting it up. Our 2FA is a software implementation, which means that you'll need an extra device such as a smart phone and an extra 2FA software to generate your TOTP (time-based one time passwords).


## Code access

You store your public SSH key with your fortrabbit Account. You can also install multiple keys with your Account, for instance one for your desktop, one for your laptop. fortrabbit automatically installs your up-to-date key(s) on each App you have access to.

### Check your SSH keys

Please revisit your list of SSH keys from time to time and keep it as short as possible. Only keep those keys you are really using.

### Passwords in the Dashboard

We save all passwords encrypted. That's why we can't read your passwords. So when you lost a password, eg for MySQL, you can only set a new one. All those passwords are randomly generated strings with high entropy.


## Firewalling

Per default all outgoing calls on all ports, except for [standard ones](/quirks#toc-firewalling), are closed. You can request to whitelist a port or port range. You do so in the Dashboard in the settings of your App.


## Vulnerability reporting

> You can't defend. You can't prevent. The only thing you can do is detect and respond.

Bruce Schneier

Do you have discovered a security issue related to fortrabbit? Please disclose in a responsible manner. We have high regard for white hat hacking culture and will work with you to understand the scope of the issue.

To report a possible security issue, please mail to: [security@fortrabbit.com](mailto:security@fortrabbit.com) using our [PGP key](/fortrabbit.pgp.asc).

**Thanks for keeping fortrabbit secure!**  Mayank Bhatodra, Salman Khan Champion [dezignburg.com](http://www.dezignburg.com), YOU?