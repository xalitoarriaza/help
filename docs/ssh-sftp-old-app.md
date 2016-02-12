---

dontList:      true
template:      article
title:         Using SSH
naviTitle:     SSH
lead:          Learn about the classical way to deploy and access your App on fortrabbit.
dontList:      true
oldApp:        true

seeAlsoLinks:
    - deployment
    - security

---

We recommend to [deploy with Git](git) to fortrabbit. But in some cases it's very helpful to access your App thru SSH (and maybe even SFTP).

## Use cases

* Dealing with static assets outside the Git repo
* Manual controlling the automated deployment workflow
* Using [Git submodules](git-submodules)
* Advanced deployments workflows
* Quick and dirty code changes and hotfixes (not recommended)
* Legacy deployment (not recommended)


## Usage

Please just use your pre-installed public key to [authenticate](security). The SSH username you enter is to identify your App:

```bash
ssh u-my-app@ssh1.eu1.frbit.com
```

#### Installed extensions

* vim, nano
* less, more, tail
* grep-family, sed, awk
* wget, curl
* tar, gzip, bzip2, zip
* git
* php
* rsync
* composer


### Executing scripts

Sometimes you need to execute scripts from SSH. Be it framework CLIs or just a simple shell script. You can execute `.sh` shell scripts and `.php` PHP scripts by calling them through the interpreter. You CANNOT execute them directly.

#### Generic example usage

Put a simple script `test.php` in your `~/htdocs` folder

```php
echo "Hello there";
```

To execute run the following

```bash
cd ~/htdocs
php test.php
```

#### Laravel artisan example usage

Many frameworks come nowadays with built-in command line tools. With Laravel, there is `artisan`. Here is how you would run it:

```bash
cd ~/htdocs
php artisan
```


### Upgrading SSH to a Worker Node

The standard fortrabbit SSH access is a shared one, so don't expect it to be very powerful. For example: you cannot run cron tasks or background workers. If you need any of those, you can upgrade to a [Worker Node](workers) plan.

### SFTP

Hello Total Commander, you want to deploy like its 1999? Well, you can do that as well. SFTP is like FTP, but different and secure. Nearly every FTP client speaks SFTP as well. Please mind that SSH key authorization is the standard authentication method on fortrabbit, most FTP clients support this as well. You can also add [username/password authentication](security) in the Dashboard.

### Rsync

Rsync is a dinosaur but still alive and kickin'. We recommend to use Git as your primary code deployment, but there still are cases in which it makes sense to to switch back to good ol' `rsync`. For example, if you need to migrate user generated runtime data. Following an example on how to sync your local `my-app/upload` folder with the corresponding one in your App: 

```bash
rsync -avp --dry-run --delete --exclude=/.git/ -e ssh my-app/upload/ u-my-app@ssh1.eu1.frbit.com:~/htdocs/upload/
```

WORD: You must remove the `--dry-run` flag to execute the synchronization. With the flag, the above command will give you only a preview on what would be done. 


#### Authorized keys file

When [logged in](directory-structure) via SSH to fortrabbit you can "see" two files:

* `.ssh/authorized_keys` — managed by you and kept for legacy reasons. We don't recommend to use it anymore.
* `.ssh/authorized_keys2` — managed by the Dashboard.

### Password authentication for SSH

For SSH access you can also enable password authentication in the Dashboard. This way you can also access your code in a legacy, classical way. It's not recommended, especially when working in a team.


### Composer on remote

Composer is also installed in your SSH account - you can also run it from the shell.