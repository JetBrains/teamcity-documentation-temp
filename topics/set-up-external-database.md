[//]: # (title: Set up External Database)
[//]: # (auxiliary-id: Set up External Database;Setting up External Database;Setting up an External Database)

<snippet id="intro">

TeamCity stores build history, users, build results, and some runtime data in a built-in SQL database. 

The currently used database is shown on the __Administration | Global Settings__ page. It is also mentioned in `teamcity-server.log` on the server startup. `HSQL*` means that the internal database is in use. For stability and security, it is recommended to migrate from the default HSQL to an external database located on a separate machine.

>If you have been evaluating TeamCity with the internal database and want to copy the data from it, read how to [migrate to an external database](migrating-to-external-database.md).
>
{style="tip"}

</snippet>

> See the full list of stored data [here](manual-backup-and-restore.md).
> 
{style="note"}

## Default Internal Database

On the first TeamCity run, using an internal HSQLDB database is suggested by default. The internal database suits evaluation purposes only; it works out of the box and requires no additional setup.

However, __we strongly recommend using an external database as a backend TeamCity database in a production environment__. An external database is usually more reliable and provides better performance: the internal database may crash and lose all your data (for example, on the "out of disk space" condition). The internal database may become extremely slow on large data sets (database storage files over 200 MB). Please note that our support does not cover any performance or database data loss issues if you are using the internal database. [Migrate to an external database](migrating-to-external-database.md) the moment you start relying on the data stored on your TeamCity server.

## Selecting External Database Engine
[//]: # (AltHead: selectingDatabase)

As a general rule, you should use the database that suits your environment best and that you can better maintain/configure in your organization. While we strive to make sure TeamCity functions equally well under all the [supported databases](supported-platforms-and-environments.md#Databases), issues may surface in some of them under high TeamCity-generated load.

You may also want to estimate the [required database capacity](system-requirements.md#Estimating+External+Database+Capacity).

## General Steps

1. Configure the external database to be used by TeamCity (see the [database-specific sections](#Database-specific+Steps) below).
2. Configure the connection to the database via the form on the [first TeamCity server start](start-teamcity-server.md) or by configuring the [database connection settings](#Database+Configuration+Properties) manually.  
    Note that TeamCity assumes ownership over the database schema. The database structure is created on the first start and actively modified during the upgrade to the new TeamCity version. The schema is not changed when TeamCity is working normally.  
    The user account used by TeamCity should have permissions to create new, modify, and delete existing tables in its schema, in addition to usual read/write permissions on all tables.
3. You may also need to download the JDBC driver for your database. Due to licensing terms, TeamCity does not bundle the driver `.jar` files for external databases. You will need to download the Java JDBC driver and put the appropriate `.jar` files (see driver-specific sections below) from it into the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/lib/jdbc` directory.    
    Note that the `.jar` files should be compiled for the Java version not later than the one used to run TeamCity. Otherwise, you might get the "_Unsupported major.minor version_" errors related to the database driver classes.

<anchor name="SettingupanExternalDatabase-DatabasespecificSteps"/>

## Database-specific Steps

The section below describes the required configuration on the database server and the TeamCity server.

### MySQL

#### On MySQL Server Side

Recommended database server settings:
 * Use InnoDB storage engine.
 * Use [utf8mb4 character set](configuring-utf8-character-set-for-mysql.md) (or utf8 if MySQL version is 5.5.2 or earlier).
 * Use case-sensitive collation.
 * Make sure that the time zone of the JVM running TeamCity and that of MySQL instance are the same using the `my.cnf` file or by configuring time zones at the OS level.
 * The server's [sql_require_primary_key](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_require_primary_key) system variable must be set to `OFF` since TeamCity manages tables without primary keys.
 * [See also recommendations for MySQL server settings](how-to.md#Configure+Newly+Installed+MySQL+Server).

The MySQL user account that will be used by TeamCity must be granted all permissions on the TeamCity database. This can be done by executing the following SQL commands from the MySQL console:

```sql
create database <database-name> collate utf8mb4_bin; -- or utf8_bin on MySQL 5.5.2 or earlier
create user <user-name> identified by '<password>';
grant all privileges on <database-name>.* to <user-name>;
grant process on *.* to <user-name>;

```

#### On TeamCity Server Side (with MySQL)

JDBC driver installation:
1. Download the MySQL [JDBC driver](http://dev.mysql.com/downloads/connector/j/). Make sure to use a version compatible with your server. 
2. For Windows, run the installer in "Custom" mode and select "MySQL Connectors | Connector/J" to install a standalone connector.
3. Copy `mysql-connector-j-<version>.jar` from the downloaded archive (Linux) or the installation folder (Windows) into the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/lib/jdbc` directory (remove the existing files there, if any). Proceed with the TeamCity setup.

### PostgreSQL

#### On PostgreSQL Server Side

1. Create an empty database for TeamCity in PostgreSQL.
   * Make sure to set up the database to use UTF8.
   * Grant permissions to modify this database to the user account used by TeamCity to work with the database.
2. [See also recommendations for PostgreSQL server settings](how-to.md#Configure+Newly+Installed+PostgreSQL+Server).

TeamCity does not specify which schema will be used for its tables. By default, PostgreSQL creates tables in the `public` [schema](https://www.postgresql.org/docs/9.1/static/ddl-schemas.html). TeamCity can also work with other PostgreSQL schemas. To switch to a different schema, create a schema named exactly as the username. This can be done using the `pgAdmin` tool or with the following SQL:

```sql
create schema teamcity authorization teamcity;

```
The schema has to be empty (it must not contain any tables).

#### On TeamCity Server Side (with PostgreSQL)

Download the required [PostgreSQL JDBC42 driver](https://jdbc.postgresql.org/download) and place it into the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/lib/jdbc` directory (remove the existing files there, if any). Proceed with the TeamCity setup.

### Oracle

#### On Oracle Server Side

Create an Oracle user account/schema for TeamCity.

>TeamCity uses the primary character set (char, varchar, clob) for storing internal text data and the national character set (nchar2, nvarchar, nclob) for storing the user input and data from external systems, like VCS and NTLM.

* Make sure that the national character set of the database instance is UTF or Unicode.
* Grant the `CREATE SESSION` and `CREATE TABLE` permissions to a user whose account will be used by TeamCity to work with this database.

<!--[//]: # (Internal note. Do not delete. "Setting up an External Databased282e338.txt")   --> 

On the first connect, TeamCity creates all the necessary tables and indices in the user's schema. (Note: TeamCity never attempts to access other schemas even if they are accessible.)

Make sure the TeamCity user has quota for accessing the table space.

#### On TeamCity Server Side (with Oracle)

1. Get the Oracle JDBC driver. Supported driver versions are 11.1 and later. The Oracle JDBC driver must be compatible with your Oracle server.  
   Place the following files:
   * `ojdbc8.jar` (or `ojdbc6.jar`, `ojdbc7.jar` depending on your database version)
   * `orai18n.jar` (can be omitted if missing in the driver version)  
   into the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/lib/jdbc` directory (remove the existing files there, if any).  
  It is strongly recommended locating the driver in your Oracle server installation. Contact your DBA for the files if required. Alternatively, download the Oracle JDBC driver from the [Oracle website](https://www.oracle.com/technetwork/database/features/jdbc/index-091264.html).
2. Proceed with the TeamCity setup.

### Microsoft SQL Server
[//]: # (AltHead: settingUpMSSQL)

For step-by-step instructions, see the [dedicated page](setting-up-teamcity-with-ms-sql-server.md). The current section provides key details required for the setup.

#### On MS SQL Server Side

>TeamCity uses the primary character set (char, varchar, text) for storing internal text data and the national character set (nchar, nvarchar, ntext) for storing the user input and data from external systems, like VCS and NTLM.

1. Create a new database. As the primary collation, use the case-sensitive collation (collation name ending with `_CS_AS`) corresponding to your locale.
2. Create a TeamCity user and ensure that this user is the owner of the database (grant the user `dbo` rights), which will give the user the ability to modify the database schema. For SSL connections, ensure that the version of MS SQL server and the TeamCity version of Java are compatible. We recommend using the latest version of the SQL server.
3. Allocate sufficient transaction log space depending on how intensively the server will be used. The recommended setup is not less than 1 GB.
4. Make sure the SQL Server browser is running.
5. Make sure the TCP/IP protocol is enabled for the SQL Server instance.

#### On TeamCity Server Side (with MS SQL)

1. Download the [Microsoft JDBC driver v12.6+](https://learn.microsoft.com/en-us/sql/connect/jdbc/release-notes-for-the-jdbc-driver?view=sql-server-ver16#126) (`sqljdbc_12.6.x` package) from the [Microsoft Download Center](https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server).
2. Unpack the downloaded package into a temporary directory. Copy the `mssql-jdbc-<version>.jre8.jar` from the just downloaded package into the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/lib/jdbc` directory (remove the existing files there, if any). MS SQL integrated security (Windows authentication) requires installing `sqljdbc_auth.dll` from the driver package as per [instructions](setting-up-teamcity-with-ms-sql-server.md#integratedSecurityAuth).
3. Proceed with the TeamCity setup.

#### jTDS Driver

It is not recommended using the jTDS JDBC driver, as it has known issues with using Unicode characters.

If you use the driver (`jtds` text appears in the `connectionUrl` of `database.properties`), it is highly recommended switching the native driver:
1. Create the server [backup](teamcity-data-backup.md) including the database.
2. Stop the server and configure the server to use the native Microsoft JDBC driver as noted in the section above.
3. Restore the database from the backup into the new MS SQL database.
4. Run the server.

## Configure the Database Connection


<anchor name="Database+Configuration+Properties"/>

### Properties File

The database connection settings are stored in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/database.properties` file. The file is a Java [properties file](https://en.wikipedia.org/wiki/.properties). You can modify it to specify required properties for your database connections.

For all supported databases there are [template files](teamcity-data-directory.md#.dist+Template+Configuration+Files) with database-specific properties located in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config` directory. The files have the `database.<database_type>.properties.dist` naming format and can be used as a reference on the required settings.

TeamCity uses Apache DBCP for database connection pooling. Refer to [Apache Commons documentation](https://commons.apache.org/dbcp/configuration.html) for detailed description of configuration properties.

### Environment Variables

The main database connection settings can be defined by setting environment variables in the TeamCity server's environment. Environment variables can be used instead of (or in addition to) the properties in the `database.properties` file.


Configuring the database connection with environment variables can be useful in the following scenarios:

* Starting the TeamCity server without defining a `database.properties` file.
* Avoiding exposure of the database password in the `database.properties` file, by setting the `TEAMCITY_DB_PASSWORD` environment variable instead.

<table>
<tr>
<th>
Variable
</th>
<th>
Description</th>
</tr>

<tr>
<td>
<code>TEAMCITY_DB_URL</code>
</td>
<td>
JDBC connection string for the database, for example:
<ul>
<li>
<code>jdbc:postgresql://localhost:5432/teamcityDB</code> 
</li>
<li>
<code>jdbc:mysql://localhost:3306/teamcityDB</code>
</li>
</ul>

</td>
</tr>

<tr>
<td>
<code>TEAMCITY_DB_USER</code>
</td>
<td>
Username for connecting to the database
</td>
</tr>

<tr>
<td>
<code>TEAMCITY_DB_PASSWORD</code>
</td>
<td>
Password for connecting to the database
</td>
</tr>
</table>

Note that environment variables take precedence over the corresponding properties in the `database.properties` file:

* `TEAMCITY_DB_URL` overrides `connectionUrl`
* `TEAMCITY_DB_USER` overrides `connectionProperties.user`
* `TEAMCITY_DB_PASSWORD` overrides `connectionProperties.password`

<seealso>
        <category ref="installation">
            <a href="common-problems.md">Common database-related problems</a>
            <a href="migrating-to-external-database.md">Migrating to an External Database</a>
        </category>
        <category ref="concepts">
            <a href="teamcity-data-directory.md">TeamCity Data Directory</a>
        </category>
        <category ref="admin-guide">
            <a href="teamcity-data-backup.md">TeamCity Data Backup</a>
        </category>
</seealso>
