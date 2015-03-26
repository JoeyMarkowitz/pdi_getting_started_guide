Easy install guide for getting a simple base install of Pentaho Community Data Integration software up and running

## Overview
This guide will get you up and running with a pentaho data integration installation so you can get to what is important quickly, building transforms!

postgres is natively supported by pentaho (easiest way to get started).  we are going to setup pentaho data integration with a test job that will run every hour and execute a test transformation.  the data integration server (carte) will link to a repository in a postgres db.  

### Pentaho Software
* spoon   - desktop app
* pan     - CLI execute transforms or export repository to xml file (local)
* kitchen - CLI execute jobs (local)
* carte   - remote execute jobs & transforms


## Install
### Setup new user in linux to run PDI
 create new user **pentaho** on the remote server to run the carte server

### install Pentaho data integration client
as of 2015.03.10 this is the download location of the pentane community version
http://community.pentaho.com/projects/data-integration/
this file extracted into a folder called data-integration

### install postgres
#### centos
  # source: https://wiki.postgresql.org/wiki/YUM_Installation
  # postgresql92 - PostgreSQL client programs and libraries
  # postgresql92-server - The programs needed to create and run a PostgreSQL server

    sudo yum install postgresql92 postgresql92-server postgresql92-contrib
    sudo service postgresql-9.2 initdb

#### ubuntu
  TBD


## Configure
### Postgres Server
#### update connection permissions 
    # File: (/var/lib/pgsql/9.2/data/pg_hba.conf)
    # add the following to the config
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            ident
    host    all             all             0.0.0.0/0   	    	md5


#### update port and listen to external ip addresses
    # File: (/var/lib/pgsql/9.2/data/postgresql.conf)
    listen_addresses = '*'          	# what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
    port = 5433                         # (change requires restart)

sudo service postgresql-9.2 restart


#### Create PDI Repository DB and postgres user pentaho
    sudo postgres
    psql
    CREATE DATABASE pdi_repository;
    CREATE USER pentaho LOGIN PASSWORD 'pentahopassword';		#defaults with login permission enabled
    GRANT ALL ON DATABASE pdi_repository TO pentaho;

    #list databases with \l;
    #list tables with \dt;
    #list roles with \du;  


    Summary Connection Info
    IP: localhost or 127.0.0.1
    PORT: 5432 (default)
    USR: pentaho
    PWD: pentahopassword
    DB: pdi_repository



### create data integration repository (in spoon on postgres db)

overview
* create db connection
* create new db in postgres called xyz
* connect to pg db and create repository
** this will create the repository tables in the database and populate the ~/.kettle/repositories.xml file
* import repository.xml from another pentaho repository if needed



in spoon
1) go to the Repository Connections Dialog (Tools > Repository > Connect or command-E)
2) create new repository (click green + in top right)
3) select the type of repository to create ("Kettle database repository")
4) create a new db connection (click NEW)
5) see screen shot
Section in top left: General
Connection Name: localhost_postgres_db
Connection Type: PostgreSQL
Host Name: localhost (ip from previous section)
Database Name: pdi_repository (from previous section)
Port Number: 5432
User Name: pentaho (from previous section)
Password: pentahopassword (from previous section)





### configure spoon
#### setup slave server connection in spoon

click slave tab
click green +
Server name: Some Slave Server Name
Hostname or IP address: localhost
Port: 8081
Web App Name (required for DI Server): BLANK
Username: cluster
Password: cluster
Is the master: checked


### setup slave server (carte)

on the remote computer where carte will run, put the configuration.xml file into ~/data-integration/configuration.xml

start slave server
./carte.sh configuration.xml


### configure carte to have access to repository
=====================================
when you created the above db connection and repository in spoon, it autogenerated a repository.xml file in ~/.kettle/repository.xml

copy ~/.kettle/repository.xml to data-integration folder on the remote server where carte will run


## Test setup
### test slave server up and running

http://localhost:8081/kettle/status/

default carte login
USR: cluster
PWD: cluster



### test transform and jobs work
Create new transform
  to keep things simple create a select now() in table input, and write that to a db with table output

create job to call transform with a start and success


default repository login
USR: admin
PWD: admin


default carte login
USR: cluster
PWD: cluster


### test jobs and transforms can run remotely on carte via spoon
# check db updated
select * from test_new_pentaho

### test jobs and transforms can run from carte server via curl or cron schedule
curl --user cluster:cluster http://localhost:8081/kettle/runJob/?job=/cron_hourly&level=Debug




Other info
.kettle/shared.xml  - shared db connections




configure carte to start with init script on system boot
==========================================================
http://techarena51.com/index.php/how-to-create-an-init-script-on-centos-6/
