---

template:     article
title:        Using Git submodules
naviTitle:    Git submodules
lead:         Leverage Git submodules on fortrabbit.
dontList:     true

tags:
    - advanced
    - git

seeAlsoLinks:
    - deployment
    - composer

externalLinks:
    - http://git-scm.com/book/en/Git-Tools-Submodules
    - http://chrisjean.com/2009/04/20/git-submodules-adding-using-removing-and-updating
    - http://joncairns.com/2011/10/how-to-use-git-submodules/

---


## Word!

This article only applies to [Old Apps](old-apps), Git Submodules are not available for New Apps. **Also in any case**: Please consider to use a Git subtree instead of a Git submodule. Git subtrees are easy to use and mostly do the job quite well.


## Usage

You can use our official [Git deployment](deployment), but Git is also installed in your [Apps SSH account](ssh-sftp-old-app). Git submodules can be used from there. Depending on your App directory structure, your Git submodules and your "root" Git repository (using those submodules) can be anywhere in the directory structure. In this example it is assumed that the submodules are used relative to `~/htdocs`.

### Generate SSH Key

This step is only required if you use a private Git repository requiring public key authentication in your submodules.

As you should not use your "regular" SSH key (the one you have used in the Dashboard) you first create a new SSH key-pair locally.  After that, copy the private key into `~/.ssh/id_rsa` when logged in via SSH on the fortrabbit remote via SSH.


### Init the git repo on SSH

You need a Git repository to work with Git submodules.

Our [Git deployment](git) does not create a Git repository in your `~/htdocs` folder and does not care whether one exists (i.e. will not delete it later on). However, you do not need to add anything to the newly created Git repository but the `.gitmodules` file!


```bash
cd ~/htdocs
git init .
git add .gitmodules
git commit -am 'Git repo for submodules'
```


### Prepare Git submodules

If you've uploaded your code via SSH, you've probably copied the submodules contents as well. In this case, clear the git submodules folder and add them again. Assuming this is what your `.gitmodules` file looks like:

```
[submodule "some-submodule"]
    path = some-submodule
    url = git@mysubmodule.tld:some-submodule.git
[submodule "other/sub"]
    path = other/sub
    url = git@otherrepo.tld:other-sub.git
```

Now is a good time to do a backup, then continue like this:

```bash
# clear the submodule directories
rm -rf some-module
rm -rf other/sub

# re-add the submodules
git submodule add git@mysubmodule.tld:some-submodule.git some-submodule
git submodule add git@otherrepo.tld:other-sub.git other/sub

# commit again
git commit -am 'Re-Added Git submodules'
```

### Update Git submodules

Later on, you can update your git submodules. Make sure, after uploading an updated `.gitmodules` file that you commit it before the update:

```bash
git commit -am 'Updated Git submodules'
git submodule update --init
```