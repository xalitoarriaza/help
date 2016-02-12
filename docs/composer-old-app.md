---

template:   article
title:      Leveraging Composer - for Old Apps
naviTitle:  Composer
lead:       Learn how to integrate Composer into your development workflow with fortrabbit.
dontList:   true
oldApp:     true
linkOther:  /composer

seeAlsoLinks:
   - deployment

tags:
   - git

keywords:
  - composerphp
  - getcomposer
  - Dependency Manager

---

No need to explain this anymore: [Composer](http://getcomposer.org) is the defacto standard to handle PHP application dependencies, as well as providing mechanisms to keep them up2date.

## Usage

On fortrabbit you [(of course) exclude](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md) the vendor folder from Git. This directory is created by Composer within in your project locally and contains all the modules you are using. This way only your own code — not the dependencies — is included into Git. During deployment Composer runs on the remote Node of your App as well. 

For [Old Apps](new-apps) you can trigger Composer with a special keyword in your commit message. For [New Apps](new-apps) Composer will always run.


### Composer via the deployment file

You can fine tune your deployment behavior and aspects of Composer in the deployment file. [V1 for Old Apps](/deployment-file-v1-old-app), 

### Composer hook trigger (only Old Apps)

Besides the recommended deployment file, you can trigger Composer on a per-commit basis for [Old Apps](new-apps). All you need to do is add `[trigger:composer:install]` to your commit message and push the commit. For example:

```bash
git commit -am 'Comments about code [trigger:composer:install]'
git push
```

KEEP IN MIND: The deployment checks only in the latest (last) commit message for the trigger directive.


### Update VS Install

In production Apps, you should always use Composer's `install` mode. The difference between install and update is:

* `install` looks in your `composer.lock` file and installs the exact versions it finds there. You can trigger it with `[trigger:composer:install]`.
* `update` checks out the `composer.json` file and applies the version patterns you've provided. You can trigger it with `[trigger:composer:update]`.

In essence: If you use `install` then you can be sure that your (last) locally installed package versions are matching exactly with the ones in your (fortrabbit) production environment. With `update` the package versions can diverge and lead to those nasty "but it works on my machine" bugs everybody loves so much. So if you are not sure: Stick with `install`.

We also support a third, legacy mode, which can be triggered by `[trigger:composer]`. It's checks whether the `vendor` folder exits and uses `update` mode if so and `install` mode if not. **We don't recommend it anymore**.

### Run the hook without code commit

```bash
# no code changes, but want to run Composer?
# empty commit to trigger Composer
git commit --allow-empty -m 'Just run composer [trigger:composer:install]'
git push
```

### Use Private repositories in Composer

You need to add the private repositories into your `composer.json` file. Here is an example, read more in the official [Composer documentation](https://getcomposer.org/doc/05-repositories.md#hosting-your-own).

```
{
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