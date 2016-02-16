---

template:       article
naviTitle:      GitHub/Bitbucket and fortrabbit
title:          Combine fortrabbit with Bitbucket or GitHub
lead:           Learn how to integrate the most popular Git-as-a-service providers with your fortrabbit workflow.
dontList:    true
oldApp:      true
linkOther:   /bitbucket-github-and-fortrabbit

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

There are two ways to use Bitbucket/GitHub with fortrabbit:

## Two remotes

Most commonly you add your external Git-provider as one remote and fortrabbit as another remote. Then you can: push to your external provider while in development and collaborating; push to fortrabbit to see the code in action.

## Deployment chain

It is also possible to create a deployment chain with Git post hooks: You push to GitHub/Bitbucket first, that trigger a hook that pushes to fortrabbit as well, then it gets deployed. This way you can also integrate testing with Continuous Integration › only deploy to fortrabbit when all test are OK.

#### Adding and and pushing to an additional remote

```bash
# go to your local projects folder
cd your-project-folder

# add fortrabbit as an additional remote to GitHub/Bitbucket
git remote add fortrabbit git@git1.eu1.frbit.com:my-app.git

# pushing to fortrabbit instead of pushing to master
git push fortrabbit

# map fortrabbit to the master branch (optional)
git config remote.fortrabbit.push refs/heads/YOUR-LOCAL-BRANCH:refs/heads/master
```


### GitHub API limits and Composer

The reason why you can run into this issue temporarily is that GitHub limits its API per IP. Given that you deploy on our Git servers (or use composer from our SSH servers) this IP address is shared with other developers deploying there as well. So if there is a deployment spike, GitHub might close down for a while.

There are two ways to circumvent the limit: the first is just to enter your github username and password when you are asked for. The second, more permanent solution, is to create a GitHub OAuth token and put it in your @composer.json@ file.

#### Create a GitHub OAuth token

Open up a terminal and issue the following command:

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

What you need is the `token` value. In the above example this is `12345abc1234512345`.

#### Modify your composer.json file

All you need to do now is add the token in the correct place in your `composer.json` file. Say it looks currently like this:


```
{
  "require": {
    "awesome/package": "@dev"
  }
}
```

All you need to do is modify it like so:

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