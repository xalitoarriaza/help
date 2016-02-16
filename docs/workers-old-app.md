---

template:     article
title:        Workers — for Old Apps
naviTitle:    Workers
lead:         Offshore long running and compute intensive tasks with background jobs — that's what Workers are for.
dontList:      true
oldApp:        true
linkOther:     /workers

tags:
    - advanced

---


## Problem

Simple truth: websites are better when faster. Generating caches, sending transactional mails, communicating with remote APIs, RSS feed reading and parsing, image processing, uploading data to external storage — don't let your users suffer.

Consider using a background job for any web request that runs longer than 500ms — which doesn't mean you don't need to optimize your code any more.

## Solution

```nohighlight
 |||||                        *********      ************
| o o |                       *       *      *          *
|  |  | <-request/response->  *  App  * <--> *  Workers *
| -_- |                       *       *      *          *
 -----                        *********      ************
```

What you want is: dedicated back-end (PHP) processes that are totally isolated from your web front-end. The Worker Node is the fortrabbit way to go:

### Implementation

SSH is automaticlly available for all our Apps — but the application is limited for common tasks like unpacking or moving around things. As a consequence our shared SSH has allows only a short process runtime and limited memory.

The Worker Node is build on top of that. If booked it replaces the general SSH access. It's a dedicated, isolated, individually scalable node, accessible via SSH — think of it as a little VPS just for this one purpose. There are various Worker Node sizes, differing in memory and CPU resources. Just book a Worker Node in the [dashboard](https://dashboard.fortrabbit.com).

#### Possible future changes

Our upcoming "Ephemeral Apps" are going to be different and the Worker will change along with it. We will probably introduce a new process (not node) based model in the future. This will introduce a more affordable entry level pricing — making it better suitable for simpler tasks.


## Usage

With your own Worker Node you can execute persistently running **worker tasks** and scheduled **cron tasks**. Manage all this is with the pre-installed command line tool the "Scheduler" in two steps:

#### 1: Configure the Scheduler YAML file
```yml
# ~/data/workers.yml
TestWorker:
    type:    worker
    script:  htdocs/artisan
    args:
        - 'queue:listen'
```

#### 2: Talk to the Scheduler CLI
```bash
scheduler setup data/workers.yml
scheduler status
# [1 Workers]
# +------------+---------+---------+--------+---------------------+---------+-----------+------+
# | Worker     | Status  | Memory  | Uptime | Started             | Stopped | Die Count | Pid  |
# +------------+---------+---------+--------+---------------------+---------+-----------+------+
# | TestWorker | started | 22.29MB | 34s    | 2013-06-25 12:10:22 | -       | -         | 7974 |
# +------------+---------+---------+--------+---------------------+---------+-----------+------+
```



### Scheduler configuration file syntax

You can name your Scheduler configuration file however you like and you can put it anywhere in the webspace of your App; you might just include it in your Git repo and push it up with everything else.

#### Layout

```yml
# the name
TheName:

    # required type (cron|worker)
    type: worker

    # required path to script, relative to home
    script: htdocs/vendor/foo/bin/worker.php

    # optional args
    args:
        - 'foo'
        - '--bar'

    # optional environment variables
    env:
        FOO: bar
        BAZ: zoing

    # optional worker startup settings
    startup:
        expect:  I have started
        timeout: 10

    # optional worker shutdown settings
    shutdown:
        signal:  SIGUSR1
        timeout: 10

    # required cron interval (if type cron)
    interval:
        minute: */5
```

#### name

**required** — needs to be unique across all tasks of any type. You can later on use it to control the tasks, by restarting it, getting status informations or alike. Allowed chars: a-z, A-Z, 0-9.

#### type

**required** — defines whether this task is a `worker` task or a `cron` task.

#### script

**required** — the script contains the relative path from the `$HOME` folder to the PHP script. Only PHP scripts are allowed. For example: if the script is in `~/htdocs/vendor/foo/bar/worker.php` the value would be `htdocs/vendor/foo/bin/worker.php`.

#### args

**optional** — array of arguments which should be passed as command line arguments to the script. Arguments may not contain line breaks (`\n` or `\r`) or quotes (`"` or `'`).

#### env

**optional** — associative array of environment variables. Variable names must consist of a-z, A-Z, 0-9 and _ and have a max length of 25 chars.


#### startup

**optional, worker only** — Allows you to control the start process of your worker tasks. You can check whether a worker task outputs an expected string. If it doesn't, the worker is assumed not to have started correctly and will be stopped again. The `expect` attribute defines a string which the worker needs to print out to be considered started successfully. The `timeout` attribute is in seconds and can range from `0` (indefinite -> expected is ignored) to `300` (5 minutes). If the process does not print out the defined string until then it will be shutdown. If startup is not defined, the worker is always assumed to started successfully unless it dies.


#### shutdown

**optional, worker only** — Structure defining how the worker is to be shutdown. Shutdowns occur when you issue a restart command, when remove the task or when you upgrade to a different plan. The `signal` attribute let you define a custom [POSIX signal](http://en.wikipedia.org/wiki/Unix_signal#POSIX_signals). Allowed are SIGQUIT, SIGKILL, SIGHUP, SIGINT, SIGTERM, SIGUSR1 and SIGUSR2. With the `timeout` attribute you can set a custom grace time in seconds until the process is terminated via SIGKILL. The time can range from 1 to 1200 (20 minutes). If signal is not set SIGQUIT is used. The timeout defaults to 300.


##### interval

**required, cron only** Structure defining the intervals / periods the cron job is call on. Each attribute can be set to a specific time (integer - 0), multiple times (comma separated integers - 0,15,30,45), an expression.

* **minute** —  ranges from 0 to 59
* **hour** —  ranges from 0 to 23
* **day of month** —  ranges from 1 to 31
* **month** —  ranges from 1 to 12
* **weekday** —  ranges from 0 to 7 whereas 0 and 7 are sunday

All times are UTC! If an attribute is omitted `*` is assumed.

**Additional examples:**

```yml
# Run every 5 minutes
TheName:
    # ..
    interval:
        minute: */5
```

```yml
# Every day at 12h (noon) minutes
TheName:
    # ..
    interval:
        minute: 0
        hour:   12
```

```yml
# Every first and fifteenth of month at 16:00h
TheName:
    # ..
    interval:
        minute: 0
        hour:   16
        day:    1,15
```

```yml
# Every Monday & Thursday, every 3 hours (starting at 00:00h)
TheName:
    # ..
    interval:
        minute:  0
        hour:    */3
        weekday: 1,4
```


### Scheduler commands

So, after you have your execution scripts and your Scheduler configuration file in place, it's time to actually run it. For this you login via SSH to your remote Worker Node:

```bash
ssh workernodeyaddayadda.frbit.eu1.net.uli
```

After you are logged in, you can talk to the Scheduler directly like so:


#### setup


```bash
# Parse the configuration file and start worker and cron tasks:
scheduler setup workers.yml
```

* **worker tasks** — newly added worker tasks will be started, removed workers will be stopped, changed workers will be restarted, all stopped worker tasks are kept stopped, but can be removed.
* **cron tasks** — will be added or removed accordingly.


#### status

```bash
scheduler status
```

#### worker tasks status output

```bash
[2 Workers]
+-------------+---------+----------+-------+---------------------+---------+-----------+------+
| Worker      | Status  | Memory  | Uptime | Started             | Stopped | Die Count | Pid  |
+-------------+---------+----------+-------+---------------------+---------+-----------+------+
| TestWorker1 | started | 22.29 MB | 34 s  | 2013-06-25 12:10:22 | -       | -         | 7974 |
| TestWorker2 | started | 22.30 MB | 34 s  | 2013-06-25 12:10:22 | -       | -         | 7971 |
+-------------+---------+----------+-------+---------------------+---------+-----------+------+
```

* **Status failed** — worker has died and could not be restarted
* **Status removing** — worker will be removed
* **Status started** — worker is running
* **Status starting** — worker will be started
* **Status stopped** — worker is stopped
* **Status stopping** — worker will be stopped
* **Memory** — current memory usage in megabytes
* **Uptime** — duration how long the task is running
* **Started** — date and time of last start
* **Stopped** — date and time of last stop
* **Die Count** — how often the task had to be restarted
* **Pid** — the unix process ID



#### cron tasks status output

```bash
[1 Crons]
+-----------+---------+----------------------------------+---------------+------------+
| Cron      | Status  | Last Run                         | Success Count | Fail Count |
+-----------+---------+----------------------------------+---------------+------------+
| TestCron1 | enabled | 2013-06-25 12:10:23 (Successful) | 1             | -          |
+-----------+---------+----------------------------------+---------------+------------+
```

* **Status disabled** — cron task is disabled and will not run
* **Status enabled** — cron task is enabled and will run
* **Last Run** — date, time and comment for last run
* **Success Count** — amount of successful runs
* **Fail Count** — amount of runs with failure



#### start

```bash
# Start a specific task:
scheduler start TaskName

# Start two specific tasks:
scheduler start TaskName1,TaskName2

# Start all cron tasks:
scheduler start CRONS

# Start all worker tasks:
scheduler start WORKERS

# Start all:
scheduler start ALL
```

If you have stopped a Task using the `stop` command, or if the Task was marked
as `failed`, you can use this command to start the Task again.

* **worker tasks** — starts previously stopped worker task
* **cron tasks** — enables previously disabled cron task

#### stop

```bash
# Stop a specific task:
scheduler stop TaskName

# Stop two specific tasks:
scheduler stop TaskName1,TaskName2

# Stop all cron tasks:
scheduler stop CRONS

# Stop all worker tasks:
scheduler stop WORKERS

# Stop all:
scheduler stop ALL
```

Stops started worker tasks & disables enabled cron tasks. Useful if you want to disable a Task temporarily. The Task will not be removed and you can use the `start` or `restart` to start the task later on.


#### tail

```bash
# Tail logs of all worker & cron tasks:
scheduler tail

# Tail logs of a specific worker or cron tasks:
scheduler tail TaskName
```

All STDERR and STDOUT output of the each Worker or Cron is logged. With this
command you can tail (show logs while they are written) them. Log files can also be found in `~/log/php/worker-out.log` respective `~/log/php/worker-err.log`.


After you are logged via SSH on the remote you can talk to the Scheduler like this:

#### clearstats

```bash
# Clear stats for a specific task
scheduler clearstats TaskName

# Clear stats for two specific task
scheduler clearstats TaskName1,TaskName2

# Clear stats for all Crons:
scheduler clearstats CRONS

# Clear stats for all Workers:
scheduler clearstats WORKERS

# Clear stats for all:
scheduler clearstats ALL
```

Resets all counters for given task(s). Useful for example, if a bug leading a Worker to die has been fixed.

#### restart

```bash
# Restart a specific task:_
scheduler restart TaskName

# Restart two specific tasks:
scheduler restart TaskName1,TaskName2

# Restart all Crons:
scheduler restart CRONS

# Restart all Workers:
scheduler restart WORKERS

# Restart all:
scheduler restart ALL
```
* **worker tasks** — starts previously stopped worker task, stops and starts running worker task.
* **cron tasks** — enables previously disabled cron task.

## Advanced usage examples

Now you have come a long way and you know most about the fortrabbit Worker Node. Don't stop now. We are just getting started!


### Queues

Ok, you got your worker and cron tasks running. Now you figure out that you probably need something else: a pipeline with which you can push jobs into your worker. Enter: queues.

Queues and worker are like bread and butter. There are [external queue providers](external-services#queues) and, of course, you can use the database - while developing.


#### Queue example

To give you an impression how easily queues can be used and to make the example more life-like, a demo worker utilizes the [IronMQ](https://packagist.org/packages/iron-io/iron_mq) library.

Somewhere in your web application, you need to push a new message on the queue to pull it later on from the worker task:

##### 1) PHP code

```php
$ironmq = new IronMQ(array(
    "token"      => 'XXXXXXXXX',
    "project_id" => 'XXXXXXXXX'
));

$ironmq->postMessage('mails', json_encode([
    'to'      => 'recp@example.tld',
    'subject' => 'hello',
    'message' => 'Lorem Ipsum'
]);
```

Now messages get created, here the worker to pull and work on them:


```php
$ironmq = new IronMQ(array(
    "token"      => 'XXXXXXXXX',
    "project_id" => 'XXXXXXXXX'
));

while (true) {
    $message = $ironmq->getMessage('mails');
    if (!$message) {
        usleep(10000);
        continue;
    }
    $mail = json_decode($message->body);
    // eg: mail($mail->to, $mail->subject, $mail->message);
    $ironmq->deleteMessage('mails', $message->id);
}
```

##### 2) Schedulder configuration file

Assuming the worker script above is saved in `~/htdocs/mail-worker.php`, here is the corresponding `scheduler` config file:

```yml
# ~/data/workers.yml
TestWorker:
    type:    worker
    script:  htdocs/mail-worker.php
```

##### 3) Scheduler CLI comand

The last step is initiate everything. Login via SSH to your Worker Node and execute:

```bash
scheduler setup data/workers.yml
```

### Scheduler + framework

Some frameworks, like Laravel 4, already come with great worker and queue tools you can use right away.

* [Laravel + artisan + scheduler](install-laravel#toc-using-the-artisan-queue)

### Worker, scheduler & no framework

Of course, there are also (MVC framework) independent tools for running background jobs like so:

* [Bernard](http://bernard.readthedocs.org/en/latest/)
* [PHP Queue](https://github.com/CoderKungfu/php-queue)

## Alternatives

Sometimes you might just want to run a small not compute intensive script. So the above described solutions might be a bit oversized. Maybe just use an [external cron job service](external-services#toc-crons) for this.



