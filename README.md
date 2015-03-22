# Easy-Pentaho-DI-Getting-Started-Guide
Easy install getting started guide to get up and running with a simple configuration


Overview
=========
This guide will get you up and running with a pentaho data integration installation so you can get to what is important quickly, building transforms!

postgres is natively supported by pentaho (easiest way to get started).  we are going to setup pentaho data integration with a test job that will run every hour and execute a test transformation.  the data integration server (carte) will link to a repository in a postgres db.  

Pentaho Software
  spoon   - desktop app
  pan     - CLI execute transforms or export repository to xml file (local)
  kitchen - CLI execute jobs (local)
  carte   - remote execute jobs & transforms


setup new user in linux to run PDI
====================================
  on the remote server to run the carte server
  create new user pentaho


install data integration client
===============================
as of 2015.03.10 this is the download location of the pentane community version
http://community.pentaho.com/projects/data-integration/
this file extracted into a folder called data-integration



install postgres
================
centos
  ###
  # INSTALL
  ###
  # source: https://wiki.postgresql.org/wiki/YUM_Installation
  # postgresql92 - PostgreSQL client programs and libraries
  # postgresql92-server - The programs needed to create and run a PostgreSQL server

  sudo yum install postgresql92 postgresql92-server postgresql92-contrib
  sudo service postgresql-9.2 initdb

  #update connection permissions (/var/lib/pgsql/9.2/data/pg_hba.conf)
	# add the following to the config
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
host    all             all             0.0.0.0/0   	    	md5


  #update postgres.conf to set the port and to listen to external ip addresses (/var/lib/pgsql/9.2/data/postgresql.conf)
  listen_addresses = '*'          	# what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
  port = 5433                           # (change requires restart)


  # configure connection permissions



  sudo service postgresql-9.2 restart


  ###
  # SETUP DATABASE
  ###
  sudo postgres
  psql
  CREATE DATABASE pdi_repository;
  CREATE USER pentaho LOGIN PASSWORD 'pentahopassword';		#defaults with login permission enabled
  GRANT ALL ON DATABASE pdi_repository TO pentaho;

  #list databases with \l;
  #list tables with \dt;
  #list roles with \du;

ubuntu
  TBD

Summary Connection Info
IP: localhost or 127.0.0.1
PORT: 5432 (default)
USR: pentaho
PWD: pentahopassword
DB: pdi_repository



create data integration repository (postgres)
===================================

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


create db connection
create new db in postgres called xyz
connect to pg db and create repository
  this will create the repository tables in the database and populate the ~/.kettle/repositories.xml file

import repository.xml



setup slave server connection in spoon
=======================================
click slave tab
click green +
Server name: Some Slave Server Name
Hostname or IP address: localhost
Port: 8081
Web App Name (required for DI Server): BLANK
Username: cluster
Password: cluster
Is the master: checked



setup slave server (carte)
==========================

on the remote computer where carte will run, put the configuration.xml file into ~/data-integration/configuration.xml

start slave server
./carte.sh configuration.xml



test slave server up and running
================================
http://localhost:8081/kettle/status/

default carte login
USR: cluster
PWD: cluster










test setup
===========

Create new transform
  to keep things simple create a select now() in table input, and write that to a db with table output

create job to call transform with a start and 
curl --user cluster:cluster http://localhost:8081/kettle/runJob/?job=/cron_hourly&level=Debug


default repository login
USR: admin
PWD: admin


default carte login
USR: cluster
PWD: cluster


run the job remotely on carte via spoon
# check db updated
select * from test_new_pentaho



configure carte to access repository
=====================================
copy ~/.kettle/repository.xml to data-integration folder on the remote server where carte will run



Other info
.kettle/shared.xml  - shared db connections




configure carte to start with init script on system boot
==========================================================
http://techarena51.com/index.php/how-to-create-an-init-script-on-centos-6/

