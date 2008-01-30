Introduction
------------

This VM image is provided in the hope that it makes OpenNMS a little easier
to evaluate.  All of the initial configuration work has been done on this VM
so that after modifying two files you should be able to start OpenNMS and
kick the tires.  It is not recommended that this VM image be used in
production except for smaller networks.

The VM
------

This VM was created with Mandriva 2008 in VMware Server 1.0.3.  This VM is
expected to be compatible with VMware Workstation 5.x and higher, VMware
Server 1.x and higher, VMware Player 2.x and higher, and VMware Fusion, but 
has not been tested on the full range of VMware products.

It was built to use 768 MB of RAM and 4GB of disk space.  It is installed
with OpenNMS 1.3.9.

The root password is "r00t".

You may want to run "urpmi --auto-update" as root to apply the latest updates
to the operating system.

Getting Started
---------------

This VM was created to make things simple.  Only two processes need to be
running for OpenNMS to work: PostgreSQL, and OpenNMS.

PostgreSQL was configured to start automatically, but you can check that it
is running with:

  ps -ef | grep postmaster

OpenNMS has a startup script in /etc/init.d.  It does not start automatically
since two files should be modified before starting OpenNMS.  They are
described below.  More general information can be found on the OpenNMS Wiki:

  http://www.opennms.org/index.php/QuickStart

discovery-configuration.xml
---------------------------

Almost all of the files used to configure OpenNMS can be found in the:

  /opt/opennms/etc

directory.  Most are XML files, which makes them both human and machine
readable.  If you haven't worked directly with XML files before, don't worry.
They are very much like other mark up files like HTML with the exception that
they aren't very tolerant of errors, so be careful when you edit them and it
is a good idea to make a backup.

The discovery process in OpenNMS is a ping sweep.  In order to tell the
application what to "ping" you need to edit the discovery-configuration.xml
file.

By default it looks like this:

<discovery-configuration threads="1" packets-per-second="1"
        initial-sleep-time="300000" restart-sleep-time="86400000"
        retries="3" timeout="800">

        <include-range retries="2" timeout="3000">
                <begin>192.168.0.1</begin>
                <end>192.168.0.254</end>
        </include-range>

</discovery-configuration>

The text inside the angle brackets ( < and > ) are called tags.  Thus the
discovery-configuration.xml file starts off with the "discovery-configuration"
tag.  The other values such as "threads" are called attributes.  The discovery
how-to has more detail, but for this README just understand that the
"include-range" and "include-url" tags represent IP addresses to be scanned.

In this example, the include range will scan the 192.168.0.0 class C network.

If your network was, say, 10.1.1.1 to 10.1.1.254, you'd want to change the
file to read:

<discovery-configuration threads="1" packets-per-second="1"
        initial-sleep-time="300000" restart-sleep-time="86400000"
        retries="3" timeout="800">

        <include-range>
                <begin>10.1.1.1</begin>
                <end>10.1.1.254</end>
        </include-range>

</discovery-configuration>

Note that the include file has been removed.

If you had another network to scan, say 172.20.1.0, you just add another
definition tag, making the file:

<discovery-configuration threads="1" packets-per-second="1"
        initial-sleep-time="300000" restart-sleep-time="86400000"
        retries="3" timeout="800">

        <include-range>
                <begin>10.1.1.1</begin>
                <end>10.1.1.254</end>
        </include-range>

        <include-range>
                <begin>172.20.1.1</begin>
                <end>172.20.1.254</end>
        </include-range>

</discovery-configuration>

snmp-config.xml
---------------

The discovery-configuration.xml file is really the only file that must be
edited before restarted OpenNMS, but unless you have all of your SNMP
community strings set to "public" you probably won't experience a lot of the
cool features of OpenNMS based on SNMP.

The snmp-config.xml contains the definitions of what community strings to use
for what devices.  The default file is pretty empty:

<snmp-config port="161" retry="3" timeout="800"
             read-community="public"
                 version="v1"
             max-vars-per-pdu="10" >

</snmp-config>

This will cause OpenNMS to try the "public" community string and to use SNMP
version 1.  In the /opt/opennms/etc/examples directory there is a better file,
but I'll post the one here from the how-to:

<snmp-config retry="3" timeout="800"
             read-community="public" write-community="private">
  <definition version="v2c">
    <specific>192.168.0.5</specific>
  </definition>

  <definition retry="4" timeout="2000">
    <range begin="192.168.1.1" end="192.168.1.254"/>
    <range begin="192.168.3.1" end="192.168.3.254"/>
  </definition>

  <definition read-community="bubba" write-community="zeke">
    <range begin="192.168.2.1" end="192.168.2.254"/>
  </definition>

</snmp-config>

As before, the file starts out with an "snmp-config" and ends with a
"/snmp-config tag".  The default attributes are 3 retries, an 800 ms timeout,
and a read community string of "public" (we have a place for a write community
string but we don't use it).

In the first definition tag we set 192.168.0.5 to use SNMP version 2c.

The second definition sets the retry to 4 and timeout to 2000ms (2 secs) for
the two class C networks 192.168.1.0 and 192.168.3.0.

The third definition sets the read community string to "bubba" for the
192.168.2.0 network (and the write community string to "zeke" although it
isn't used).

Set the community strings for your network in this file.  To anticipate the
question, you can only associate one read community string per IP address.
This is to minimize SNMP authentication failure traps.

Starting It All Up
------------------

After you have modified those files, run:

  /sbin/service opennms start

to start OpenNMS.

If it returns "OK" you're good to go.  Point your browser to

  http://[opennms VM Address]:8980/opennms

where [opennms VM Address] is the hostname or IP address for the VM, and

  username: admin
  password: admin

Getting Help
------------

If OpenNMS does not start, you may have an error in the files you modified.
Go to:

  /var/log/opennms

and run:

  grep -E '(ERROR|FATAL)' *

Look in discovery.log and capsd.log for errors.

You can also get help from the OpenNMS mailing lists:

  http://www.opennms.org/index.php/Mailing_lists

and commercial support is available from The OpenNMS Group:

  http://www.opennms.com

Good luck!

