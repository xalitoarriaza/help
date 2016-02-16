---

template:      article
title:         All about MySQL
naviTitle:     MySQL
lead:          Access and configure the database.
dontList:      true
oldApp:        true
linkOther:     /mysql

keywords:
     - localhost
     - mysqldump
     - dump
     - mysql
     - database

seeAlsoLinks:
    - ssh-sftp-old-app
    - scaling

---


PHP + MySQL is a classic. fortrabbit offers MySQL as an internal component for each App. Its use is optional and it comes in different sizes, you can [scale](scaling) it up and down any time without downtimes.


## Access MySQL from your App

The fortrabbit Dashboard provides you with the credentials you will need to provide your App access. Usually you have some kind of configuration file where you enter those credentials.



#### MySQL config example

```php
'mysql' => array(
    'host'      => 'appname.mysql.eu1.frbit.com',
    'database'  => 'appname',
    'username'  => 'appname',
    'password'  => 'etGHteshH',
    'driver'    => 'mysql',
    'charset'   => 'utf8',
    'collation' => 'utf8'
)
```

PRO TIP: Use environment detection to differ between your local environment and the one on fortrabbit.


## Remote MySQL access

Sometimes you want to run a certain query on your live database. Or you want to dump your database. So you need to access the MySQL database on fortrabbit remotely.

For security reasons you can not connect to the MySQL database from "outside". But you can open a [SSH tunnel](http://en.wikipedia.org/wiki/Tunneling_protocol) and then connect to the MySQL database thru this tunnel.

### MySQL GUIs

Sometimes it's handy to manage your MySQL database with a graphical interface.

#### Native (Desktop) App

You can use a graphical interface like the official [MySQL Workbench](http://www.mysql.com/products/workbench/) (Mac/Linux/Windows). Most of those clients have connection presets that help you to establish the SSH tunnel and the MySQL connection in one convenient step.

#### Browser based

[phpMyAdmin](http://www.phpmyadmin.net/) and [Chive](http://www.chive-project.com/) are the most common choices. It's of course possible to install either of them on any fortrabbit App, but for security reasons we recommend not to do so.

Use phpMyAdmin or Chive locally instead. Once you have setup the [SSH tunnel](#toc-shell-tunnel-mysql) you can connect to the remote database as if it were local.

### Terminal › Tunnel › MySQL

If you are familiar with the shell then this is no biggie. In essence: open up a tunnel to your App's MySQL database using your Apps SSH account.


```bash
# open the tunnel on local port 13306
ssh -N -L 13306:my-app.mysql.eu1.frbit.com:3306 u-my-app@ssh1.eu1.frbit.com
```

`13306` is an arbitrary port. Choose something in the higher range (10000-65000). This command will not reply with any message on success! If nothing shows up, you did right!

Once the tunnel is up, you can connect to your MySQL database with the `mysql` console client **from another terminal**:

```bash
mysql -umy-app -h127.0.0.1 -P13306 -p my-app
```

You will be asked for password: enter your MySQL password in this step!

WORD: Use `127.0.0.1`, not `localhost` as the database host!

## Advanced use-cases

###  Export & import

A common task is to move your MySQL data around, e.g. if you are migrating to fortrabbit or you are about to set up a staging environment. GUIs have easy to use export/import wizards, in the terminal you can do this like so:


#### mysqldump & mysql

Using `mysqldump` and `mysql` is the standard approach to migrate a database between two MySQL servers. First of start by exporting your data from your old database (example):

```bash
# on your local machine or on the old server
mysqldump database-name > database.sql
```

Now [create a tunnel](#toc-shell-tunnel-mysql) to your fortrabbit App's MySQL database and import everything:

```bash
mysql -h127.0.0.1 -P13306 -umy-app -p my-app < database.sql
```

#### LOAD DATA

You can export and import a large, single table with the following example:

```bash
# on your local machine or on the old server
echo 'SELECT * FROM tablename;' | mysql database-name > tablename.sql
```

Now import everything via a tunnel to yourfortrabbit MySQL database and import

```bash
mysql --local-infile=1 -h127.0.0.1 -P13306 -umy-app -p my-app

# on the mysql shell
mysql> LOAD DATA LOCAL INFILE '/path/to/tablename.sql' INTO TABLE tablename;
```



### Different time zone

MySQL has [time zone support](http://dev.mysql.com/doc/refman/5.5/en/time-zone-support.html) Our nodes default to the standard time zone "UTC". If you want to change this time zone, you can do so on a "per connection" basis.

There are two approaches to tackle this issue: handle the time zone on application level or handle the time zone on database level. Each has its merits and which one is better strongly depends on the use case. This article shows you how to set the time zone in the database.

#### Setting time zone in plain (My)SQL

The syntax to change the time zone is:

```sql
-- set time zone to +3 hours
SET time_zone = '+03:00';

-- set time zone to -7 hours
SET time_zone = '-07:00';
```

You can query the current time zone like so:

```sql
SELECT @@session.time_zone;
```

#### Setting time zone with PDO

`PDO` offers configurable options when setting up the connection. One of them allows you to issue commands right after initialization (connection).

```php
$pdo = new PDO(
    'mysql:host=my-app.mysql.eu1.frbit.com;dbname=my-app',
    'my-app',
    'the-password-string',
    array(
        PDO::MYSQL_ATTR_INIT_COMMAND => 'SET time_zone = \'+02:00\''
    )
);
```

#### Setting time zone in Laravel

As Eloquent uses `PDO`, you can use the `PDO::MYSQL_ATTR_INIT_COMMAND` option. Extend your `mysql` configuration array in `app/config/database.php` or your specific environment `database.php` file:

```php
return array(
    // ..
    'mysql' => array(
        'driver'    => 'mysql',
        'host'      => 'my-app.mysql.eu1.frbit.com',
        'database'  => 'my-app',
        'username'  => 'username',
        'password'  => 'the-password-string',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
        'options'   => array(
            // Here is the time zone setting
            \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET time_zone = \'+02:00\''
        )
    ),
    // ..
);
```

#### Setting time zone in Symfony

Doctrine uses as well `PDO` for MySQL connections, you can set the `PDO::MYSQL_ATTR_INIT_COMMAND` in your configuration. Assuming YAML configuration, you would do this in `app/config/config.yml`:

```yml
doctrine:
    dbal:
        driver:   %database_driver%
        host:     %database_host%
        port:     %database_port%
        dbname:   %database_name%
        user:     %database_user%
        password: %database_password%
        charset:  UTF8
        options:
            1002: "SET time_zone = '+02:00'"
```

The `1002` is the value of `PDO::MYSQL_ATTR_INIT_COMMAND`, as you cannot use PHP constants in a YAML file.




## Alternatives

For certain use cases other databases or object storage might be an interesting alternative, you can integrate those over an [external service](external-services).