# Send Rate Policyd

This simple daemon help you to limit the outbound rate of Postfix servers. Uses MySQL to store per user
or domain rate limits

For INSTALL doc refer to http://www.simonecaruso.com/limit-sender-rate-in-postfix/

## Create Table

See: `emails.sql`

~~~sql
CREATE TABLE  `emails` (
    `email` VARCHAR(65) NOT NULL,
    `messagequota` INT(10) UNSIGNED NOT NULL DEFAULT '0',
    `messagetally` INT(10) UNSIGNED NOT NULL DEFAULT '0',
    `timestamp` INT(10) UNSIGNED DEFAULT NULL,
    );
~~~

## perl module dependancy
Switch.pm - 

## Postfix send rate limit per user/domain

I needed to limit the number of emails sent by users through your Postfix server, and store message quota in RDMS systems (maybe you have multiple MX).

I tried policyd or other policy daemons for Postfix;  but they all miss support for SQL storage.

With this piece of code you can setup a send rate per users or sender domain on daily/weekly/monthly basis and store data in MySQL (PostgreSQL,…).

To work with mqdaemon you only need a mysql table described above.

and setup configuration parameters (`db_table`, `db_wherecol`, `db_messagequota`, etc) inside the Perl script.

You must insert all rows for the emails you have to enforce! The script is configured to use a simple UPDATE.

Modify the postfix data restriction class “smtpd_data_restrictions” like the following:

~~~
smtpd_data_restrictions = check_policy_service inet:$IP:$PORT
~~~

To print the cache content with update statistics for username into log file just send a SIGHUP to the process PID using `kill -HUP $pid` or invoke the script with `printshm` paramenter (`perl ./daemon.pl printshm`).

To enforce per-domain quota set the search key (`$s_key_type` parameter) to `domain`.

Start the daemon with `./daemon.pl` and nill it with `pkill daemon.pl`.

Take care of using a port higher than 1024 to run the script as non-root.

This daemon caches the quota in memory, so you don’t need to worry about I/O operations.
