# Easy-Pentaho-DI-Getting-Started-Guide
Easy install getting started guide to get up and running with a simple configuration






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
  # source: https://wiki.postgresql.org/wiki/YUM_Installation
  # postgresql92 - PostgreSQL client programs and libraries
  # postgresql92-server - The programs needed to create and run a PostgreSQL server

  sudo yum install postgresql92 postgresql92-server postgresql92-contrib
  sudo service postgresql-9.2 initdb

ubuntu
  TBD


create data integration repository (postgres)
===================================
postgres is natively supported by pentaho (easiest way to get started)

create db connection
create new db in postgres called xyz
connect to pg db and create repository
  this will create the repository tables in the database and populate the ~/.kettle/repositories.xml file

import repository.xml



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



setup slave server connection in spoon
=======================================
command-E to enter repository
click slave tab
click green +
Server name: Some Slave Server Name
Hostname or IP address: localhost
Port: 8081
Web App Name (required for DI Server): BLANK
Username: cluster
Password: cluster
Is the master: checked






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

