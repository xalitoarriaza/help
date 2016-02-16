---

template:      article
title:         All about MySQL
naviTitle:     MySQL
lead:          PHP + MySQL is a classic. Access and configure the most common database on fortrabbit.
linkOther:     /mysql-old-app

keywords:
     - localhost
     - mysqldump
     - dump
     - mysql
     - database
     - innodb
     - myisam

seeAlsoLinks:
    - ssh-sftp-old-app
    - scaling

---

## Booking & scaling

MySQL is implemented as an Component. It's use is optional and it comes in different sizes. You can [scale](scaling#toc-mysql) it up and down any time without downtimes in the Dashboard.


## Access MySQL from your App

Usually you have some kind of configuration file in which you enter those credentials. fortrabbit provides the access credentials for MySQL via the [App secrets](secrets). Here is a generic example on how to access them:

```php
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

return [
    'mysql' => [
        'host'      => $secrets['MYSQL']['HOST'],
        'database'  => $secrets['MYSQL']['DATABASE'],
        'username'  => $secrets['MYSQL']['USER'],
        'password'  => $secrets['MYSQL']['PASSWORD'],
        'driver'    => 'mysql',
        'charset'   => 'utf8',
        'collation' => 'utf8'
    ]
];
```

PRO TIP: Use environment detection to differ between your local environment and the one on fortrabbit.

## Remote MySQL access

Sometimes you want to run a certain query on your live database. Or you want to dump your database. So you need to access the MySQL database on fortrabbit remotely.

For security reasons you can not connect to the MySQL database from "outside". But you can open a [SSH tunnel](http://en.wikipedia.org/wiki/Tunneling_protocol) and then connect to the MySQL database thru this tunnel.

### Obtain your credentials

To access MySQL from remote you still need the MySQL username, password, host and so on. You can get those using an [SSH command](secrets#toc-accessing-app-secrets).

### MySQL GUIs

Sometimes it's handy to manage your MySQL database with a graphical interface. We recommend the official [MySQL Workbench](http://www.mysql.com/products/workbench/) (Mac/Linux/Windows) from Oracle. There is also [Navicat](http://www.navicat.com/products/navicat-for-mysql) (also multi-platform), [HeidiSQL](http://www.heidisql.com/) for Windows and [Sequel Pro](http://www.sequelpro.com/) for Mac. And a [host of others](https://www.google.com/search?q=mysql%20gui).

Most of those clients have connection presets that help you to establish the SSH tunnel and the MySQL connection in one convenient step.

### Shell › Tunnel › MySQL

If you are familiar with the shell then this is no biggie. In essence: Open up a tunnel to your App's MySQL database:

```bash
# open the tunnel on local port 13306
ssh -N -L 13306:my-app.mysql.eu2.frbit.com:3306 tunnel@tunnel.eu2.frbit.com
```

`13306` is an arbitrary port. Choose something in the higher range (10000-65000).

**This command will not reply with any message on success! If nothing shows up: you did right!** *This behavior is how most (all?) SSH clients are implemented and sadly we cannot issue any response message telling you that it worked. So mind: if you see no error, all is good.*

Once the tunnel is up, you can connect to your MySQL database with the `mysql` console client **from another terminal**:

```bash
mysql -umy-app -h127.0.0.1 -P13306 -p my-app
```

You will be asked for password: enter your MySQL password in this step! Use `127.0.0.1`, not `localhost` as the database host!


##  Export & import

A common task is to move your MySQL data around, e.g. if you are migrating to fortrabbit or you are about to set up a staging environment. GUIs have easy to use export/import wizards, in the terminal you can do this like so:

### mysqldump & mysql

Using `mysqldump` and `mysql` is the standard approach to migrate a database between two MySQL servers. First of start by exporting your data from your old database (example):

```bash
# on your local machine or on the old server
mysqldump database-name > database.sql
```

Now [create a tunnel](#toc-shell-tunnel-mysql) to your fortrabbit App's MySQL database and import everything:

```bash
mysql -h127.0.0.1 -P13306 -umy-app -p my-app < database.sql
```

### LOAD DATA

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
$secrets = json_decode(file_get_contents($_SERVER['APP_SECRETS']), true);

$pdo = new PDO(
    'mysql:host='. $secrets['MYSQL']['HOST']. ';dbname='. $secrets['MYSQL']['DATABASE'],
    $secrets['MYSQL']['USER'],
    $secrets['MYSQL']['PASSWORD'],
    array(
        PDO::MYSQL_ATTR_INIT_COMMAND => 'SET time_zone = \'+02:00\''
    )
);
```

## Alternatives

For certain use cases other databases or object storage might be an interesting alternative, you can integrate those over an [external service](external-services).