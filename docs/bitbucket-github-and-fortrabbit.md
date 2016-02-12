---

template:       article
naviTitle:      GitHub/Bitbucket and fortrabbit
title:          Combine fortrabbit with Bitbucket or GitHub
lead:           Learn how to integrate the most popular Git-as-a-service providers with your fortrabbit workflow.
linkOther:   /bitbucket-github-and-fortrabbit-old-app

seeAlsoLinks:
    - external-services
    - deployment
    - git-submodules

tags:
    - advanced

keywords:
    - addon
    - add-on
    - integrations
    - integration
    - API
    - CI
    - cloud

---

It's actually quite simple: add your external Git-provider as one remote and fortrabbit as another remote. Then you can push to your external provider while in development and collaborating; push to fortrabbit to see the code in action.

Here is how you add your App's Git remote on fortrabbit to your already existing local working copy of a GitHub / Bitbucket repo and then work with it:

```bash
# add your App's remote and name it "fortrabbit"
$ git remote add fortrabbit git@deploy.eu2.frbit.com:my-app.git

# now push to the master branch of your App's remote on fortrabbit
$ git push fortrabbit master
```

## GitHub API limits and Composer

The reason why you can run into this issue temporarily is that GitHub limits it's API per IP. Given that you deploy on our Git servers (or use composer from our SSH servers) this IP address is shared with other developers deploying there as well. So if there is a deployment spike, GitHub might close down for a while.

The solution is to create a GitHub OAuth token and put it in your `composer.json` file. Open up a terminal and issue the following command:

```bash
curl -u your-github-user -d '{"note": "Fortrabbit Auth"}' https://api.github.com/authorizations
```

This will give you a response like the following:

```
{
  "id": 123456,
  "url": "https://api.github.com/authorizations/123456",
  "app": {
    "name": "Fortrabbit Auth (API)",
    "url": "http://developer.github.com/v3/oauth/#oauth-authorizations-api",
    "client_id": "12345abc12345"
  },
  "token": "12345abc1234512345",
  "note": "Fortrabbit Auth",
  "note_url": null,
  "created_at": "2013-08-08T11:08:18Z",
  "updated_at": "2013-08-08T11:08:18Z",
  "scopes": []
}
```

What you need is the `token` value. In the above example it is `12345abc1234512345`. Open up your `composer.json` file and add the token withunder the `config` directive:

```
{
  "require": {
    "awesome/package": "@dev"
  },
  "config": {
    "github-oauth": {
      "github.com": "12345abc1234512345"
    }
  }
}
```

That's it. Your API token will be used to give you a personal rate limit which is much more higher than the default one.