% POSTGRESQL-95(1) Container Image Pages
% SoftwareCollections.org
% July 19, 2017

PostgreSQL 9.5 SQL Database Server Docker image
===============================================

This container image includes PostgreSQL 9.5 SQL database server for OpenShift and general usage.
Users can choose between RHEL and CentOS based images.
The RHEL image is available in the [Red Hat Container Catalog](https://access.redhat.com/containers/#/registry.access.redhat.com/rhscl/postgresql-95-rhel7)
as registry.access.redhat.com/rhscl/postgresql-95-rhel7.
The CentOS image is then available on [Docker Hub](https://hub.docker.com/r/centos/postgresql-95-centos7/)
as centos/postgresql-95-centos7.


Description
-----------

This container image provides a containerized packaging of the PostgreSQL postgres daemon
and client application. The postgres server daemon accepts connections from clients
and provides access to content from PostgreSQL databases on behalf of the clients.
You can find more information on the PostgreSQL project from the project Web site
(https://www.postgresql.org/).


Usage
-----

For this, we will assume that you are using the `rhscl/postgresql-95-rhel7` image.
If you want to set only the mandatory environment variables and not store the database
in a host directory, execute the following command:

```
$ docker run -d --name postgresql_database -e POSTGRESQL_USER=user -e POSTGRESQL_PASSWORD=pass -e POSTGRESQL_DATABASE=db -p 5432:5432 rhscl/postgresql-95-rhel7
```

This will create a container named `postgresql_database` running PostgreSQL with
database `db` and user with credentials `user:pass`. Port 5432 will be exposed
and mapped to the host. If you want your database to be persistent across container
executions, also add a `-v /host/db/path:/var/lib/pgsql/data` argument. This will be
the PostgreSQL database cluster directory.

If the database cluster directory is not initialized, the entrypoint script will
first run [`initdb`](http://www.postgresql.org/docs/9.5/static/app-initdb.html)
and setup necessary database users and passwords. After the database is initialized,
or if it was already present, [`postgres`](http://www.postgresql.org/docs/9.5/static/app-postgres.html)
is executed and will run as PID 1. You can stop the detached container by running
`docker stop postgresql_database`.



Environment variables and volumes
---------------------------------

The image recognizes the following environment variables that you can set during
initialization by passing `-e VAR=VALUE` to the Docker run command.

**`POSTGRESQL_USER`**  
       User name for PostgreSQL account to be created

**`POSTGRESQL_PASSWORD`**  
       Password for the user account

**`POSTGRESQL_DATABASE`**  
       Database name

**`POSTGRESQL_ADMIN_PASSWORD`**  
       Password for the `postgres` admin account (optional)


The following environment variables influence the PostgreSQL configuration file. They are all optional.

**`POSTGRESQL_MAX_CONNECTIONS (default: 100)`**  
       The maximum number of client connections allowed

**`POSTGRESQL_MAX_PREPARED_TRANSACTIONS (default: 0)`**  
       Sets the maximum number of transactions that can be in the "prepared" state. If you are using prepared transactions, you will probably want this to be at least as large as max_connections

**`POSTGRESQL_SHARED_BUFFERS (default: 32M)`**  
       Sets how much memory is dedicated to PostgreSQL to use for caching data

**`POSTGRESQL_EFFECTIVE_CACHE_SIZE (default: 128M)`**  
       Set to an estimate of how much memory is available for disk caching by the operating system and within the database itself


You can also set the following mount points by passing the `-v /host:/container` flag to Docker.

**`/var/lib/pgsql/data`**  
       PostgreSQL database cluster directory


**Notice: When mouting a directory from the host into the container, ensure that the mounted
directory has the appropriate permissions and that the owner and group of the directory
matches the user UID or name which is running inside the container.**


PostgreSQL auto-tuning
--------------------

When the PostgreSQL image is run with the `--memory` parameter set and if there
are no values provided for `POSTGRESQL_SHARED_BUFFERS` and
`POSTGRESQL_EFFECTIVE_CACHE_SIZE` those values are automatically calculated
based on the value provided in the `--memory` parameter.

The values are calculated based on the
[upstream](https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server)
formulas. For the `shared_buffers` we use 1/4 of given memory and for the
`effective_cache_size` we set the value to 1/2 of the given memory.

PostgreSQL admin account
------------------------
The admin account `postgres` has no password set by default, only allowing local
connections.  You can set it by setting the `POSTGRESQL_ADMIN_PASSWORD` environment
variable when initializing your container. This will allow you to login to the
`postgres` account remotely. Local connections will still not require a password.


Changing passwords
------------------

Since passwords are part of the image configuration, the only supported method
to change passwords for the database user (`POSTGRESQL_USER`) and `postgres`
admin user is by changing the environment variables `POSTGRESQL_PASSWORD` and
`POSTGRESQL_ADMIN_PASSWORD`, respectively.

Changing database passwords through SQL statements or any way other than through
the environment variables aforementioned will cause a mismatch between the
values stored in the variables and the actual passwords. Whenever a database
container starts it will reset the passwords to the values stored in the
environment variables.


Upgrading database (by switching to newer PostgreSQL image version)
-------------------------------------------------------------------

** Warning! Please, before you decide to do the data directory upgrade, always
ensure that you've carefully backed up all your data and that you are OK with
potential manual rollback! **

This image supports automatic upgrade of data directory created by
the PostgreSQL server version 9.4 (and _only_ this version) - provided by sclorg
image.  The upgrade process is designed so that you should be able to just
switch from *image A* to *image B*, and set the `$POSTGRESQL_UPGRADE` variable
appropriately to explicitly request the database data transformation.

The upgrade process is internally implemented via `pg_upgrade` binary, and for
that purpose the container needs to contain two versions of PostgreSQL server
(have a look at `man pg_upgrade` for more info).

For the `pg_upgrade` process - and the new server version, we need to initialize
a brand new data directory.  That's data directory is created automatically by
container tooling under /var/lib/pgsql/data, which is usually external
bind-mountpoint.  The `pg_upgrade` execution is then similar to dump&restore
approach -- it starts both old and new PostgreSQL servers (within container) and
"dumps" the old datadir while and at the same time it "restores" it into new
datadir.  This operation requires a lot of data files copying, so you can decide
what type of upgrade you'll do by setting `$POSTGRESQL_UPGRADE` appropriately:

|    Variable value  |    Description                                 |
| :----------------- | ---------------------------------------------- |
|  `copy`            | The data files are copied from old datadir to new datadir.  This option has low risk of data loss in case of some upgrade failure. |
|  `hardlink`        | Data files are hard-linked from old to the new data directory, which brings performance optimization - but the old directory becomes unusable, even in case of failure. |

Note that because we copy data directory, you need to make sure that you have
enough space for the copy;  upgrade failure because of not enough space might
lead to data loss.


Troubleshooting
---------------
At first the postgres daemon writes its logs to the standard output, so these are available in the container log. The log can be examined by running:

    docker logs <container>

Then log output is redirected to logging collector process and will appear in directory "pg_log".


See also
--------
Dockerfile and other sources for this container image are available on
https://github.com/sclorg/postgresql-container.
In that repository, Dockerfile for CentOS is called Dockerfile, Dockerfile
for RHEL is called Dockerfile.rhel7.
