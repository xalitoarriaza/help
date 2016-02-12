---

template:      article
naviTitle:     Git
title:         Git on fortrabbit
lead:          Git is the popular version control system for code. Learn which Git features to expect on fortrabbit.

seeAlsoLinks:
    - deployment
    - bitbucket-github-and-fortrabbit
    - dashboard
    - app
    - terminology
    - 

tags: 
    - beginner

---

We get support requests regarding Git usage on fortrabbit on a regular basis. This article shall help to get started and solve common misunderstandings.


## Get ready for Git

To use Git you need to install it locally on your development machine. And you also need should be familiar with it (commit, push, pull, …). It's not very intuitive at start, but very handy when you know it a bit better. There are many good tutorials out there in the interwebs to get started.


## Using Git as version control

Each [App](app) comes with a dedicated Git remote repo, it is required to use Git with fortrabbit. The obvious benefits of using a version control system are:

* A history of all files included, so you can undo all changes
* Powerful file merging on line base which makes collaboration easy


## Git is not GitHub

[GitHub](https://guthub.com) is a popular provider of hosted Git repositories. Sometimes people confuse Git with GitHub. GitHub has extended Git workflows with neat communication tools around the basic Git usage. Most notable is the "pull request" workflow. The fortrabbit [Dashboard](dashboard) does not offer such project communication tools — it's barebone Git only. 

You can however build your [own workflow](bitbucket-github-and-fortrabbit) which includes a hosted Git repo on GitHub or Bitbucket (another Git repo provider).


## Using Git for deployment

Apart from the standard usage we also use Git to deploy your code. Your ``git push`` on the master branch will update the remote repository and trigger a chain of processes which will finally distribute your code changes to your public [Apps](app). Please see our [deployment article](deployment) for detailed informations.




