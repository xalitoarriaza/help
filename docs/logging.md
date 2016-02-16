---

template:   article
title:      Acces your App's logs
naviTitle:  Logging
lead:       Accessing live logs of your App is essential for developing. Here is how you can do it on fortrabbit.

keywords:
    - Logging
    - Logs

tags:
    - beginner

---

## Problem

You are developing your App and see the white screen of death. You are getting a 5xx error and don't know why. You write debug logs and need them to trace a problem.

## Solution

Use the SSH logging command of your App to get a live stream of all the logs:

```bash
# Per default all sources are tailed together:
$ ssh log@log.eu2.frbit.com tail app-name

# Only Apache access log, all incoming requests with response status, time-stamp, additional headers and the first line of the request:
$ ssh log@log.eu2.frbit.com tail app-name source:apache_access

# Only Apache error log, which can be very helpful to debug `.htaccess` files or the like:
$ ssh log@log.eu2.frbit.com tail app-name source:apache_error

# Only PHP error logs, which contain whatever your App writes on `error_log()`:
$ ssh log@log.eu2.frbit.com tail app-name source:web_php_error

# Only web standard error output, which containins everything written by your App to `STDERR`:
$ ssh log@log.eu2.frbit.com tail app-name source:web_stderr

# Only Cron Job or Nonstop Job output
$ ssh log@log.eu2.frbit.com tail app-name source:worker
```

**Hint**: Use the `mono` flag to force monochrome output, if your console displays the colors incorrectly: `ssh log@log.eu2.frbit.com tail app-name mono`.

**Hint 2**: You can use multiple `source:name` parameters at once.