Kanboard - SQLite2MySQL
=======================

Guidelines to migrate a [Kanboard](https://github.com/fguillot/kanboard) SQLite database to a MySQL / MariaDB one.

Requirements
------------------------

* SQLite3
* A [Kanboard](https://github.com/fguillot/kanboard) instance (to generate the MySQL schema)
* Zip (for backup script only, otherwize, manually backup your instance)

This script is generating a SQL data dump compatible with MySQL. The resulting SQL data dump file can be used to import data into a MySQL database. This script can also do that automatically.

This tool has been tested on GNU/Linux Debian Jessie (SQLite 2.8.17, MariaDB 10.0.23) and GNU/Linux Debian Stretch (SQLite 3.11, MySQL 14.14).
It should also work on Windows (with Cygwin or Mysys), with MariaDB, maybe Postgres.

* [Version 1.1.1 of this tool](https://github.com/oliviermaridat/kanboard-sqlite2mysql/releases/tag/v1.1.1) has been tested to migrate successfuly to Kanboard 1.1.1 (SQLite schema version 116 to MySQL schema version 126) with or without plugins.
* [Version 1.0.34 of this tool](https://github.com/oliviermaridat/kanboard-sqlite2mysql/releases/tag/v1.1.1) has been tested to migrate successfuly to Kanboard 1.0.34 with or without plugins.
* [Version 1.0.27 of this tool](https://github.com/oliviermaridat/kanboard-sqlite2mysql/releases/tag/v1.1.1) has been tested to migrate successfuly to Kanboard 1.0.27 with or without plugins.

Usage
------------------------

**/!\ Please, please, PLEASE, BACKUP YOUR before to do anything else. TEST THIS SCRIPT IN A DUMMY KANBOARD WITH A DUMMY MYSQL DATABASE AND VERIFY THE RESULT before to do anything in your production environment! You have been advised;-)**

Since the [latest version of this tool ](https://github.com/oliviermaridat/kanboard-sqlite2mysql/releases/latest) has only been tested until Kanboard 1.1.1, if you are running a Kanboard instance lower than 1.1.1, **it is recommanded to update your SQLite Kanboard instance to 1.1.1, make the conversion to MySQL using this script, and then finish the update to the last version of Kanboard**. If you are already running a Kanboard instance higher than 1.1.1, let's give it a try! Any success or failure feedback for version higher than 1.1.1 is really welcome! Thanks.

First get kanboard-sqlite2mysql:

    git clone https://github.com/oliviermaridat/kanboard-sqlite2mysql
    cd kanboard-sqlite2mysql
    chmod u+x kanboard-sqlite2mysql.sh
    chmod u+x kanboard-backup.sh
    
Then backup your Kanboard data:

    ./kanboard-backup.sh <Kanboard instance physical path>

And finally create the SQL dump file compatible with MySQL:

    ./kanboard-sqlite2mysql.sh <Kanboard instance physical path> -o db-mysql.sql

Or you can also directly apply it to the MySQL database of your choice:

    ./kanboard-sqlite2mysql.sh <Kanboard instance physical path> <MySQL DB name> [ -h <MySQL DB host> -u <MySQL DB user> -p ]

Running this script may take 2 or 3 minutes.

Did it failed? Check for [manual solutions](https://github.com/oliviermaridat/kanboard-sqlite2mysql/labels/Solution) or read the technical details below.
Anyway, its now time to create a new ticket to say if it worked or failed, to precise which version of SQLite / MySQL / Kanboard / OS your were using. Thank you!

Steps to reproduce
------------------------

Migrate from a SQLite to a MySQL or MariaDB database is not straightforward and really not in Kanboard. This is due to:

* SQL dumps of SQLite and MySQL are not exactly the same, some adjustments may be required. Several scripts can be found on the Web to do that automatically.
* SQLite does not support foreign keys but MySQL do. Therefore, even if SQLite can create a SQL dump, it will create it with SQL tables ordered alphabeticaly, and MySQL will not accept to import this in this order. E.g. if table A depends on table B, SQLite will export table A then table B, but MySQL will required to import table B and then table C...
* SQLite and MySQL schemas have diverged in Kanboard
 * Some columns have not been removed in the SQLite schema (due to missing "DROP COLUMN" feature in SQLite)
 * The order of the columns is not the same

The main idea of this script is:
* Generate a SQL dump file (compatible with MySQL) with only Kanboard data (i.e. "INSERT INTO" queries)
* Use a Kanboard instance to create the table in a MySQL database
* Import this SQL dump file into the MySQL database  

All required steps are listed below:
* Create an SQL dump from the SQLite database, for example using `sqlite3 data/db.sqlite .dump > db-mysql.sql`. Actually, this script is using something more complex in order to reorder tables.
* Change this SQL dump to something working with MySQL using a script (to be done)
 * Replace \ by /
 * Only for actions and activities replace \ by \\\\
 * Eventually add quotes (\`) around "settings" table 'option' and 'value' fields, and "columns" table name
* Either disable foreign key check (`SET FOREIGN_KEY_CHECKS = 0;`) or move  at the beginning of the dump the "INSERT INTO" queries for tables projects, columns, links, groups, users, tasks, task_has_links, subtasks, comments and actions.
* Remove data generated by Kanboard in some tables

    SET FOREIGN_KEY_CHECKS = 0;
    TRUNCATE TABLE settings;
    TRUNCATE TABLE users;
    TRUNCATE TABLE links;
    TRUNCATE TABLE plugin_schema_versions;
    SET FOREIGN_KEY_CHECKS = 1;
    
* Add temporarily (and remove afterwards) missing columns in the MySQL database
 * users: is_admin, default_project_id, is_admin_project
 
    ALTER TABLE users ADD COLUMN is_admin INT DEFAULT 0;
    ALTER TABLE users ADD COLUMN default_project_id INT DEFAULT 0;
    ALTER TABLE users ADD COLUMN is_project_admin INT DEFAULT 0;
    
    ALTER TABLE users DROP COLUMN is_admin;
    ALTER TABLE users DROP COLUMN default_project_id;
    ALTER TABLE users DROP COLUMN is_project_admin;
    
 * tasks: estimate_duration, actual_duration, 

    ALTER TABLE tasks ADD COLUMN estimate_duration VARCHAR(255) DEFAULT '';
    ALTER TABLE tasks ADD COLUMN actual_duration VARCHAR(255) DEFAULT '';
    
    ALTER TABLE tasks DROP COLUMN estimate_duration;
    ALTER TABLE tasks DROP COLUMN actual_duration;
 * project_has_users: id, is_owner

    ALTER TABLE project_has_users ADD COLUMN id INT DEFAULT 0;
    ALTER TABLE project_has_users ADD COLUMN is_owner INT DEFAULT 0;
    
    ALTER TABLE project_has_users DROP COLUMN id;
    ALTER TABLE project_has_users DROP COLUMN is_owner;
    
    
License
--------------------------------

This script is distributed under the LGPL3+ license.

Please feel free to contribute.

If you have any ideas, critiques, suggestions or whatever you want to call it, please open an issue. I'll be happy to hear from you what you'd see in this tool.


