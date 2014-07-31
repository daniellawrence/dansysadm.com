Title: Using hostinfo to generate nagios configuration
Date: 2012-02-01 10:20
Category: blog


One of the best thing ( and worst thing...) is that it uses a plain text files for its configurations.
However this is a massive advantage to the system that I was previous using, 
a mess of 150-200 tables mysql tables and an application that was closed source and no api.

With nagios you are able to use the __conf.d__ style to keep all your configuration files in one place.
You can also separated the configuration files into different files where needed in the same directory.

I have taken advantage of this to have a directory structure similar to the following.

- /usr/local/nagios/etc/
 - hostinfo
  - servers.conf
  - commands.conf
  - servicegroups.conf
  - hostgroups.conf
  - etc

All my generated configuration files are in one location however it is very easy to view the configuration files if your looking for a certain thing.

Introduction to hostinfo
------------------------

Hostinfo (<http://code.google.com/p/hostinfo/>) is a niffy key storage system written in python with a mysql backend.

Once you have it setup you use *sos-reports* and *Solaris explorers* to populate the hostinfo database.
An example of the items extracted from sos-reports and explorers are the following:

- number of cpu
- operating system
- running applications
- disks (size, type, filesystems)
- etc

Then when its all setup the magic of hostinfo is in the easy of asking it questions via the command line.

__Print out all of your systems running solaris__

```sh
  $ hostinfo os=solaris
```

__list of all systems that have a */database* filesystem__

```sh
  $ hostinfo localfs=/database
```

__list of all linux servers and the kernel version that are running apache that are in the dmz.__

```sh
  $ hostinfo -p uname os=linux apps=apache logical_location=dmz
```



Introduction to nagios configuration
-------------------------------------

Nagios configuration is all done by flat files, and overview can be found at <http://nagios.sourceforge.net/docs/3_0/configobject.html>

an example configuration file for a host would look something like this...

```sh
define host{
    host_name               bogus-router
    alias                   Bogus Router #1
    address                 192.168.1.254
    parents                 server-backbone
    check_command           check-host-alive
    check_interval          5
    retry_interval          1
    max_check_attempts      5
    check_period            24x7
    process_perf_data       0
    contact_groups          router-admins
    notification_interval   30
    notification_period     24x7
    notification_options    d,u,r
}
```

As you can see the configuration is very simple, the problem is it is very verbose. So editing this by hand would be a nightmare if you had more then 1 server and 10 checks on a server.

Generation of configuration files
---------------------------------

By running a hostinfo query to find all the hosts that I know about then running them over a simple loop similar to the below.
I can generate the hosts.cfg for nagios for 1000's of hosts in a few seconds.

```python
>>> from nagios.objects import Host
>>> test_host = Host(host_name="localhost", check_command="host_check")
>>> print test_host.cfg()
define host {
  use                                     linux-server
  check_command                           host_check
  contact_groups                          opsunix
  alias                                   localhost
  notes_url                               http://opswiki/wiki/index.php/Host:localhost
  host_name                               localhost
  address                                 127.0.0.1  
}
>>>
```

When the __Host()__ object is created it takes the hostname and looks up the details in hostinfo to work out.

- The Operating system, then common template is applied
- Monitoring times ( 24x7, 8x5, etc)
- Finds out the team responsible for the server, list that as the contact group
- Find the ip address

The object also has some static things that it will set, such as

- A link to the wiki for that host
- Setting the check command


I do the same thing with contacts by grabbing a list of people from AD that are in the __contact__ group.

```python
>>> from nagios.objects import Contact
>>> test_contact = Contact(contact_name="dansysadm")
>>> print test_contact.cfg()
define contact {
  host_notification_period                24x7
  service_notification_options            o,w,c,r
  host_notification_commands              host_notification_command
  service_notification_period             24x7
  contact_name                            dansysadm
  service_notification_commands           service__notification_command
  host_notification_options               u,d,r
  }
>>>
```

The same process is used to generate:

- services
- service groups
- host groups
- etc

The advantage
-------------

When ever a new server is build, it is added into hostinfo then the hourly refresh of nagios configuration files adds the host into nagios.

- No missing hosts from monitoring
- No missing servers from monitoring
- No one edits nagios cfg files
- When a new service is installed on a host, it will automatically be monitored!
- Its python!
