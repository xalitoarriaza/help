---

template:      article
title:         Install WordPress - for Old App
naviTitle:     WordPress
lead:          This is how to run WordPress successfully on fortrabbit — maybe.
dontList:      true
oldApp:        true
linkOther:     /install-wordpress

seeAlsoLinks:
    - app
    - install-wordpress

keywords:
    - WP
    - wpadmin

tags:
    - beginner
    - php
    - install

---

We all love it, we all hate it: our good ol' friend WordPress.

### Problem

WordPress — at least up until version 4 — is not designed to be very compatible with [Git](git)/[Composer](composer) deployment work-flows. WordPress is a legacy.


### Solutions

Sorry, there is not one thing that can be one to fix everything.
Wait! (https://github.com/roots/bedrock) looks promising,  but we haven't tried it yet.


#### Just use SFTP

For "persistent Apps" (current App version) we offer [SFTP access](ssh-sftp-old-app#toc-sftp) — so you can "upload" your code. It works. MIND: this workflow migt change in the future.

#### Just put everything in Git

Although bad practice — usually you should only put your own source code in Git  — you can put your whole WordPress installation under version control and then manage everything this way. Take care of updates!

#### Git excludes

In any case of using WordPress with Git, exclude the runtime data, put `wp-content/uploads`in your `.gitignore` file.

#### WordPress web updates

WordPress can update itself on the remote server alone. That feature is quite nice, because it helps people a lot to keep their WordPress up-to-date — which is important for [security](security) concerns.

That very feature can also turn into a nightmare when you are trying to keep your local version and the production one in sync. WordPress updates include database updates. That means the updater script actually needs to run to change the database — so you can't just push the new files up to the remote.