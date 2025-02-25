# mongodb puppet module

[![Build Status](https://github.com/voxpupuli/puppet-mongodb/workflows/CI/badge.svg)](https://github.com/voxpupuli/puppet-mongodb/actions?query=workflow%3ACI)
[![Release](https://github.com/voxpupuli/puppet-mongodb/actions/workflows/release.yml/badge.svg)](https://github.com/voxpupuli/puppet-mongodb/actions/workflows/release.yml)
[![Puppet Forge](https://img.shields.io/puppetforge/v/puppet/mongodb.svg)](https://forge.puppetlabs.com/puppet/mongodb)
[![Puppet Forge - downloads](https://img.shields.io/puppetforge/dt/puppet/mongodb.svg)](https://forge.puppetlabs.com/puppet/mongodb)
[![Puppet Forge - endorsement](https://img.shields.io/puppetforge/e/puppet/mongodb.svg)](https://forge.puppetlabs.com/puppet/mongodb)
[![Puppet Forge - scores](https://img.shields.io/puppetforge/f/puppet/mongodb.svg)](https://forge.puppetlabs.com/puppet/mongodb)
[![License](https://img.shields.io/github/license/voxpupuli/puppet-mongodb.svg)](https://github.com/voxpupuli/puppet-mongodb/blob/master/LICENSE)

#### Table of Contents

1. [Overview](#overview)
2. [Module Description - What does the module do?](#module-description)
3. [Setup - The basics of getting started with mongodb](#setup)
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
6. [Limitations - OS compatibility, etc.](#limitations)
7. [Development - Guide for contributing to the module](#development)

## Overview

Installs MongoDB on RHEL/Ubuntu/Debian from OS repo, or alternatively from
MongoDB community/enterprise repositories.

## Module Description

The MongoDB module manages mongod server installation and configuration of the
mongod daemon. For the time being it supports only a single MongoDB server
instance, without sharding functionality.

The MongoDB module also manages Ops Manager setup and the mongdb-mms daemon.

## Setup

### What MongoDB affects

* MongoDB package.
* MongoDB configuration files.
* MongoDB service.
* MongoDB client.
* MongoDB sharding support (mongos)
* MongoDB apt/yum repository.
* Ops Manager package.
* Ops Manager configuration files.

### Beginning with MongoDB

If you just want a server installation with the default options you can run
`include mongodb::server`. If you need to customize configuration
options you need to do the following:

```puppet
class {'mongodb::server':
  port    => 27018,
  verbose => true,
}
```

For Red Hat family systems, the client can be installed in a similar fashion:

```puppet
class {'mongodb::client':}
```

Note that for Debian/Ubuntu family systems the client is installed with the
server. Using the client class will by default install the server.

If one plans to configure sharding for a Mongo deployment, the module offer
the `mongos` installation. `mongos` can be installed the following way :

```puppet
class {'mongodb::mongos' :
  configdb => ['configsvr1.example.com:27018'],
}
```

Although most distros come with a prepacked MongoDB server, you may prefer to
use a more recent version. To install MongoDB from the community repository:

```puppet
class {'mongodb::globals':
  manage_package_repo => true,
  version             => '3.6',
}
-> class {'mongodb::client': }
-> class {'mongodb::server': }
```

If you don't want to use the MongoDB software repository or the OS packages,
you can point the module to a custom one.
To install MongoDB from a custom repository:

```puppet
class {'mongodb::globals':
  manage_package_repo => true,
  repo_location => 'http://example.com/repo'
}
-> class {'mongodb::server': }
-> class {'mongodb::client': }
```

Having a local copy of MongoDB repository (that is managed by your private modules)
you can still enjoy the charms of `mongodb::params` that manage packages.
To disable managing of repository, but still enable managing packages:

```puppet
class {'mongodb::globals':
  manage_package_repo => false,
  manage_package      => true,
}
-> class {'mongodb::server': }
-> class {'mongodb::client': }
```

## Usage

Most of the interaction for the server is done via `mongodb::server`. For
more options please have a look at [mongodb::server](#class-mongodbserver).
Also in this version we introduced `mongodb::globals`, which is meant more
for future implementation, where you can configure the main settings for
this module in a global way, to be used by other classes and defined resources.
On its own it does nothing.

### Create MongoDB database

To install MongoDB server, create database "testdb" and user "user1" with password "pass1".

```puppet
class {'mongodb::server':
  auth => true,
}

mongodb::db { 'testdb':
  user          => 'user1',
  password_hash => 'a15fbfca5e3a758be80ceaf42458bcd8',
}
```
Parameter 'password_hash' is hex encoded md5 hash of "user1:mongo:pass1".
Unsafe plain text password could be used with 'password' parameter instead of 'password_hash'.

### Ops Manager

To install Ops Manager and have it run with a local MongoDB application server do the following:

```puppet
class {'mongodb::opsmanager':
  opsmanager_url        => 'http://opsmanager.yourdomain.com'
  mongo_uri             => 'mongodb://yourmongocluster:27017,
  from_email_addr       => 'opsmanager@yourdomain.com',
  reply_to_email_addr   => 'replyto@yourdomain.com',
  admin_email_addr      => 'admin@yourdomain.com',
  $smtp_server_hostname => 'email-relay.yourdomain.com'
}
```

The default settings will not set useful email addresses. You can also just run `include mongodb::opsmanager`
and then set the emails later.

## Ops Manager Usage

Most of the interaction for the server is done via `mongodb::opsmanager`. For
more options please have a look at [mongodb::opsmanager](#class-mongodbopsmanager).

## Reference

### Classes

#### Public classes
* `mongodb::server`: Installs and configure MongoDB
* `mongodb::client`: Installs the MongoDB client shell (for Red Hat family systems)
* `mongodb::globals`: Configure main settings in a global way
* `mongodb::mongos`: Installs and configure Mongos server (for sharding support)
* `mongodb::opsmanager`: Installs and configure Ops Manager

#### Private classes
* `mongodb::repo`: Manage MongoDB software repository
* `mongodb::repo::apt`: Manage Debian/Ubuntu apt MongoDB repository
* `mongodb::repo::yum`: Manage Redhat/CentOS yum MongoDB repository
* `mongodb::server::config`: Configures MongoDB configuration files
* `mongodb::server::install`: Install MongoDB software packages
* `mongodb::server::service`: Manages service
* `mongodb::client::install`: Installs the MongoDB client software package
* `mongodb::mongos::config`: Configures Mongos configuration files
* `mongodb::mongos::install`: Install Mongos software packages
* `mongodb::mongos::service`: Manages Mongos service

#### Class: mongodb::globals
*Note:* most server specific defaults should be overridden in the `mongodb::server`
class. This class should only be used if you are using a non-standard OS or
if you are changing elements such as `version` or `manage_package_repo` that
can only be changed here.

This class allows you to configure the main settings for this module in a
global way, to be used by the other classes and defined resources. On its
own it does nothing.

##### `server_package_name`
This setting can be used to override the default MongoDB server package
name. If not specified, the module will use whatever package name is the
default for your OS distro.

##### `service_name`
This setting can be used to override the default MongoDB service name. If not
specified, the module will use whatever service name is the default for your OS distro.

##### `service_provider`
This setting can be used to override the default MongoDB service provider. If
not specified, the module will use whatever service provider is the default for
your OS distro.

##### `service_status`
This setting can be used to override the default status check command for
your MongoDB service. If not specified, the module will use whatever service
name is the default for your OS distro.

##### `mongod_service_manage`
This setting can be used to override the default management of the mongod service.
By default the module will manage the mongod process.

##### `mongos_service_manage`
This setting can be used to override the default management of the mongos service.
By default the module will manage the mongos process.

##### `user`
This setting can be used to override the default MongoDB user and owner of the
service and related files in the file system. If not specified, the module will
use the default for your OS distro.

##### `group`
This setting can be used to override the default MongoDB user group to be used
for related files in the file system. If not specified, the module will use
the default for your OS distro.

##### `ipv6`
This setting is used to configure MongoDB to turn on ipv6 support. If not specified
and ipv6 address is passed to MongoDB bind_ip it will just fail.

##### `bind_ip`
This setting can be used to configure MonogDB process to bind to and listen
for connections from applications on this address. If not specified, the
module will use the default for your OS distro.
*Note:* This value should be passed as an array.

##### `use_enterprise_repo`
When `manage_package_repo` is set to true, this setting indicates if it will
use the Community Edition (false, the default) or the Enterprise one (true).

##### `version`
The version of MonogDB to install/manage. This is needed when managing
repositories. If not specified, the module will use the default for your OS
distro.

##### `repo_location`
This setting can be used to override the default MongoDB repository location.
If not specified, the module will use the default repository for your OS distro.

##### `repo_proxy`
This will allow you to set a proxy for your repository in case you are behind a corporate firewall. Currently this is only supported with yum repositories

##### `proxy_username`
This sets the username for the proxyserver, should authentication be required

##### `proxy_password`
This sets the password for the proxyserver, should authentication be required

#### Class: mongodb::server

Most of the parameters manipulate the mongod.conf file.

For more details about configuration parameters consult the
[MongoDB Configuration File Options](http://docs.mongodb.org/manual/reference/configuration-options/).

##### `ensure`
Used to ensure that the package is installed and the service is running, or that the package is absent/purged and the service is stopped. Valid values are true/false/present/absent/purged.

##### `config`
Path of the config file. If not specified, the module will use the default
for your OS distro.

##### `dbpath`
Set this value to designate a directory for the mongod instance to store
it's data. If not specified, the module will use the default for your OS distro.

##### `dbpath_fix`
Set this value to true if you want puppet to recursively manage the permissions
of the files in the dbpath directory.  If you are using the default dbpath, this
should probably be false. Set this to true if you are using a custom dbpath. The
default is false.

##### `pidfilepath`
Specify a file location to hold the PID or process ID of the mongod process.
If not specified, the module will use the default for your OS distro.

##### `manage_pidfile`
Should puppet create the pidfile. Mondod 6.2.10 will not start if pidfile exists

##### `logpath`
Specify the path to a file name for the log file that will hold all diagnostic
logging information. Unless specified, mongod will output all log information
to the standard output.

##### `ipv6`
This setting has to be true to configure MongoDB to turn on ipv6 support. If not specified
and ipv6 address is passed to MongoDB bind_ip it will just fail.

##### `bind_ip`
Set this option to configure the mongod or mongos process to bind to and listen
for connections from applications on this address. If not specified, the module
will use the default for your OS distro. Example: bind_ip=['127.0.0.1', '192.168.0.3']
*Note*: bind_ip accepts an array as a value.

##### `logappend`
Set to true to add new entries to the end of the logfile rather than overwriting
the content of the log when the process restarts. Default: True

##### `system_logrotate`
Set to reopen for mongo to close a log file then reopen it so that logrotations handled
outside of mongo perform as expected. Default: undef

##### `fork`
Set to true to fork server process at launch time. The default setting depends on
the operating system.

##### `port`
Specifies a TCP port for the server instance to listen for client connections.
Default: 27017

##### `journal`
Set to true to enable operation journaling to ensure write durability and
data consistency. Default: on 64-bit systems true and on 32-bit systems false

##### `nojournal`
Set nojournal = true to disable durability journaling. By default, mongod
enables journaling in 64-bit versions after v2.0.
Default: on 64-bit systems  false and on 32-bit systems true

*Note*: You must use journal to enable journaling on 32-bit systems.

##### `smallfiles`
Set to true to modify MongoDB to use a smaller default data file size.
Specifically, smallfiles reduces the initial size for data files and
limits them to 512 megabytes.  Default: false

##### `cpu`
Set to true to force mongod to report every four seconds CPU utilization
and the amount of time that the processor waits for I/O operations to
complete (i.e. I/O wait.) Default: false

##### `auth`
Set to true to enable database authentication for users connecting from
remote hosts. If no users exist, the localhost interface will continue
to have access to the database until you create the first user.
Default: false

##### `noauth`
Disable authentication. Currently the default. Exists for future compatibility
 and clarity.

##### `verbose`
Increases the amount of internal reporting returned on standard output or in
the log file generated by `logpath`. Default: false

##### `verbositylevel`
MongoDB has the following levels of verbosity: v, vv, vvv, vvvv and vvvvv.
Default: None

##### `objcheck`
Forces the mongod to validate all requests from clients upon receipt to ensure
that clients never insert invalid documents into the database.

##### `quota`
Set to true to enable a maximum limit for the number of data files each database
can have. The default quota is 8 data files, when quota is true. Default: false

##### `quotafiles`
Modify limit on the number of data files per database. This option requires the
`quota` setting. Default: 8

##### `diaglog`
Creates a very verbose diagnostic log for troubleshooting and recording various
errors.  Valid values: 0, 1, 2, 3 and 7.
For more information please refer to [MongoDB Configuration File Options](http://docs.mongodb.org/manual/reference/configuration-options/).

##### `directoryperdb`
Set to true to modify the storage pattern of the data directory to store each
database’s files in a distinct folder. Default: false

##### `profile`
Modify this value to changes the level of database profiling, which inserts
information about operation performance into output of mongod or the
log file if specified by `logpath`.

##### `maxconns`
Specifies a value to set the maximum number of simultaneous connections
that MongoDB will accept. Default: depends on system (i.e. ulimit and file descriptor)
limits. Unless set, MongoDB will not limit its own connections.

##### `oplog_size`
Specifies a maximum size in megabytes for the replication operation log
(e.g. oplog.) mongod creates an oplog based on the maximum amount of space
available. For 64-bit systems, the oplog is typically 5% of available disk space.

##### `nohints`
Ignore query hints. Default: None

##### `nohttpinterface`
Set to true to disable the HTTP interface. This command will override the rest
and disable the HTTP interface if you specify both. Default: false

##### `noscripting`
Set noscripting = true to disable the scripting engine. Default: false

##### `notablescan`
Set notablescan = true to forbid operations that require a table scan. Default: false

##### `noprealloc`
Set noprealloc = true to disable the preallocation of data files. This will shorten
the start up time in some cases, but can cause significant performance penalties
during normal operations. Default: false

##### `nssize`
Use this setting to control the default size for all newly created namespace
files (i.e .ns). Default: 16

##### `mms_token`
MMS token for mms monitoring. Default: None

##### `mms_name`
MMS identifier for mms monitoring. Default: None

##### `mms_interval`
MMS interval for mms monitoring. Default: None

##### `configsvr`
Use this setting to enable config server mode for mongod.

##### `shardsvr`
Use this setting to enable shard server mode for mongod.

##### `replset`
Use this setting to configure replication with replica sets. Specify a replica
set name as an argument to this set. All hosts must have the same set name.

##### `replset_members`
An array of member hosts for the replica set.
Mutually exclusive with `replset_config` param.

##### `replset_config`
A hash that is used to configure the replica set.
Mutually exclusive with `replset_members` param.

```puppet
class mongodb::server {
  replset        => 'rsmain',
  replset_config => { 'rsmain' => { ensure  => present, settings => { heartbeatTimeoutSecs => 15, getLastErrorModes => { ttmode => { dc => 1 } } }, members => [{'host'=>'host1:27017', 'tags':{ 'dc' : 'east'}}, { 'host' => 'host2:27017'}, 'host3:27017']  }  }

}
```

##### `config_data`
A hash to allow for additional configuration options
to be set in user-provided template.


##### `rest`
Set to true to enable a simple REST interface. Default: false

##### `quiet`
Runs the mongod or mongos instance in a quiet mode that attempts to limit the
amount of output. This option suppresses : "output from database commands, including drop, dropIndexes, diagLogging, validate, and clean", "replication activity", "connection accepted events" and "connection closed events".
Default: false

> For production systems this option is **not** recommended as it may make tracking
problems during particular connections much more difficult.

##### `slowms`
Sets the threshold for mongod to consider a query “slow” for the database profiler.
Default: 100 ms

##### `keyfile`
Specify the path to a key file to store authentication information. This option
is only useful for the connection between replica set members. Default: None

##### `key`
Specify the key contained within the keyfile. This option
is only useful for the connection between replica set members. Default: None

##### `master`
Set to true to configure the current instance to act as master instance in a
replication configuration. Default: False  *Note*: deprecated – use replica sets

##### `set_parameter`
Specify extra configuration file parameters (i.e.
textSearchEnabled=true). Default: None

##### `syslog`
Sends all logging output to the host’s syslog system rather than to standard
output or a log file. Default: None
*Important*: You cannot use syslog with logpath. Set logpath to false to disable it.

##### `slave`
Set to true to configure the current instance to act as slave instance in a
replication configuration. Default: false
*Note*: deprecated – use replica sets

##### `only`
Used with the slave option, only specifies only a single database to
replicate. Default: <>
*Note*: deprecated – use replica sets

##### `source`
Used with the slave setting to specify the master instance from which
this slave instance will replicate. Default: <>
*Note*: deprecated – use replica sets

##### `ssl`
Set to true to enable ssl. Default: <>
*Important*: You need to have ssl_key set as well, and the file needs to
pre-exist on node. If you wish to use certificate validation, ssl_ca must also
be set.

##### `ssl_key`
Default: <>

##### `ssl_ca`
Default: <>

##### `ssl_weak_cert`
Set to true to disable mandatory SSL client authentication
Default: False

##### `ssl_invalid_hostnames`
Set to true to disable fqdn SSL cert check
Default: False

##### `ssl_mode`
Ssl authorization mode. Valid options are: requireSSL, preferSSL, allowSSL.
Default: requireSSL

##### `service_manage`
Whether or not the MongoDB service resource should be part of the catalog.
Default: true

##### `storage_engine`
Only needed for MongoDB 3.x versions, where it's possible to select the
'wiredTiger' engine in addition to the default 'mmapv1' engine. If not set, the
config is left out and mongo will default to 'mmapv1'.
You should not set this for MongoDB versions < 3.x

##### `restart`
Specifies whether the service should be restarted on config changes. Default: 'true'

##### `create_admin`
Allows to create admin user for admin database.
Redefine these parameters if needed:

##### `admin_username`
Administrator user name

##### `admin_password`
Administrator user password

##### `admin_auth_mechanism`
Administrator authentication mechanism.
scram_sha_256 password synchronization verification is not supported.
Default: 'scram_sha_1'

##### `admin_update_password`
Update password.
Used with SCRAM-SHA-256 because password verification is not supported.
Default: false

##### `admin_roles`
Administrator user roles

##### `store_creds`
Store admin credentials in mongorc.js file. Uses with `create_admin` parameter

##### `handle_creds`
Set this to false to avoid having puppet handle .mongorc.js in case you wish to deliver it by other means.
This is needed for facts to work if you have auth set to true.  Default is true.


#### Class: mongodb::mongos
class. This class should only be used if you want to implement sharding within
your mongodb deployment.

This class allows you to configure the mongos daemon (responsible for routing)
on your platform.

##### `ensure`
Used to ensure that the package is installed and the service is running, or that the package is absent/purged and the service is stopped. Valid values are true/false/present/absent/purged.

##### `config`
Path of the config file. If not specified, the module will use the default
for your OS distro.

##### `config_content`
Config content if the default doesn't match one needs.

##### `config_template`
Path to the config template if the default doesn't match one needs.

##### `config_data`
Hash containing key-value pairs to allow for additional configuration options to be set in user-provided template.

##### `configdb`
Array of the config servers IP addresses the mongos should connect to.

##### `service_manage`
Whether or not the MongoDB sharding service resource should be part of the catalog.
Default: true

##### `service_name`
This setting can be used to override the default Mongos service name. If not
specified, the module will use whatever service name is the default for your OS distro.

##### `service_provider`
This setting can be used to override the default Mongos service provider. If
not specified, the module will use whatever service provider is the default for
your OS distro.

##### `service_status`
This setting can be used to override the default status check command for
your Mongos service. If not specified, the module will use whatever service
name is the default for your OS distro.

##### `service_enable`
This setting can be used to specify if the service should be enable at boot

##### `service_ensure`
This setting can be used to specify if the service should be running

##### `package_ensure`
This setting can be used to specify if puppet should install the package or not

##### `package_name`
This setting can be used to specify the name of the package that should be installed.
If not specified, the module will use whatever service name is the default for your OS distro.

##### `restart`
Specifies whether the service should be restarted on config changes. Default: 'true'

### Definitions

#### Definition: mongodb:db

Creates database with user. Resource title used as database name.

##### `user`
Name of the user for database

##### `password_hash`
Hex encoded md5 hash of "$username:mongo:$password".
For more information please refer to [MongoDB Authentication Process](http://docs.mongodb.org/meta-driver/latest/legacy/implement-authentication-in-driver/#authentication-process).

##### `password`
Plain-text user password (will be hashed)

##### `auth_mechanism`
Authentication mechanism.
Can be either 'scram_sha_1' or 'scram_sha_256'.
scram_sha_256 password synchronization verification is not supported.
Default: 'scram_sha_1'

##### `update_password`
Update password.
Used with SCRAM-SHA-256 because password verification is not supported.
Default: false

##### `roles`
Array with user roles as string.
Roles will be granted to user's database if no alternative database is explicitly defined.
Example: ['dbAdmin', 'readWrite@other_database']
Default: ['dbAdmin']

### Providers

#### Provider: mongodb_database
'mongodb_database' can be used to create and manage databases within MongoDB.

```puppet
mongodb_database { testdb:
  ensure   => present,
  tries    => 10,
  require  => Class['mongodb::server'],
}
```
##### `tries`
The maximum amount of two second tries to wait MongoDB startup. Default: 10


#### Provider: mongodb_user
'mongodb_user' can be used to create and manage users within MongoDB database.

*Note:* if replica set is enabled, replica initialization has to come before
any user operations.

```puppet
mongodb_user { testuser:
  name          => 'testuser',
  ensure        => present,
  password_hash => mongodb_password('testuser', 'p@ssw0rd'),
  database      => testdb,
  roles         => ['readWrite', 'dbAdmin'],
  tries         => 10,
  require       => Class['mongodb::server'],
}
```
##### `username`
Name of the mongodb user.

##### `password_hash`
Hex encoded md5 hash of "$username:mongo:$password". Only available on MongoDB 3.0 and later.

##### `password`
Plaintext password of the user.

##### `database`
Name of database. It will be created, if not exists.

##### `roles`
Array with user roles as string.
Roles will be granted to user's database if no alternative database is explicitly defined.
Example: ['dbAdmin', 'readWrite@other_database']
Default: ['dbAdmin']

##### `tries`
The maximum amount of two second tries to wait MongoDB startup. Default: 10

#### Provider: mongodb_replset
'mongodb_replset' can be used to create and manage MongoDB replicasets.

```puppet
mongodb_replset { rsmain:
  ensure  => present,
  members => ['host1:27017', 'host2:27017', 'host3:27017']
}
```

Ideally the ```mongodb_replset``` resource will be declared on the initial
desired primary node (arbitrarily the first of the list) and this node will be
processed once the secondary nodes are up. This will ensure all the nodes are
in the first configuration of the replicaset, else it will require running
puppet again to add them.

##### `members`
Array of 'host:port' of the replicaset members.

It currently only adds members without options.

#### Provider: mongodb_shard
'mongodb_shard' can be used to create and manage MongoDB shards.
*Note:* Removing a shard is not yet supported. Shard can only be added.

```puppet
mongodb_shard { 'rsmain':
  member => 'rsmain/host1:27017',
  keys   => [{'rsmain.foo' => {'name' => 1}}],
}
```
##### `member`
Member of the shard in the form;

* [hostname]
* [hostname]:[port]
* [replica-set-name]/[hostname]
* [replica-set-name]/[hostname]:port

##### `keys`
Sharding keys for a specific database. This variable should be an array
of sharding keys.

## Limitations

This module has been tested on:

* Debian 7.* (Wheezy)
* Debian 6.* (squeeze)
* Ubuntu 12.04.2 (precise)
* Ubuntu 10.04.4 LTS (lucid)
* RHEL 5/6/7
* CentOS 5/6/7

For a full list of tested operating systems please have a look at the [.nodeset.xml](https://github.com/voxpupuli/puppet-mongodb/blob/master/.nodeset.yml) definition.

This module should support `service_ensure` separate from the `ensure` value on `Class[mongodb::server]` but it does not yet.

### Apt module support

While this module supports both 1.x and 2.x versions of the puppetlabs-apt module, it does not support puppetlabs-apt 2.0.0 or 2.0.1.

## Development

This module is maintained by [Vox Pupuli](https://voxpupuli.org/). Voxpupuli
welcomes new contributions to this module, especially those that include
documentation and rspec tests. We are happy to provide guidance if necessary.

Please see [CONTRIBUTING](.github/CONTRIBUTING.md) for more details.

### Authors

* Puppetlabs Module Team
* Voxpupuli Team

We would like to thank everyone who has contributed issues and pull requests to this module.
A complete list of contributors can be found on the
[GitHub Contributor Graph](https://github.com/voxpupuli/puppet-mongodb/graphs/contributors)
for the [puppet-mongodb module](https://github.com/voxpupuli/puppet-mongodb).
