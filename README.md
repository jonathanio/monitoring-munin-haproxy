# haproxyng Munin Plugin

This is a Munin plugin which allows you to monitor a large number of statistics
from within HAPRoxy, including:

  * Bandwidth for Frontends
      * Broken down by Service
  * Bandwidth for Backends
      * Broken down by Service
          * Broken down by Host
  * Timings for Backends
      * Broken down by Service
          * Broken down by Host
  * Sessions for Frontends
      * Broken down by Service
  * Sessions for Backends
      * Broken down by Service
          * Broken down by Host
  * Response Codes for Frontends
      * Broken down by Service (aggregating by Status Code)
          * Broken down by Status Code
  * Response Codes for Backends
      * Broken down by Service (aggregating by Server)
          * Broken down by Server
      * Broken down by Service (aggregating by Status Codes)
          * Broken down by Status (aggregating by Server)
              * Broken down by Server

## Warning

This can product a lot of graphs: With about 50 servers and around 10 services
you can probably exceed over 1000 graphs being produced covering all the above.

This setup either required some *very* fast hardware, or it's best to make sure
that you are using something like `rrdcached` and SSDs to make sure that you
are aggregating updates, caching reads, etc. Additionally, creating graphs
on-the-fly via CGI may end up being better, unless you can safely produce a very
large number of graphs every five minutes with room to spare and grow!

## Usage

You will need Munin 2.0 as this is a `multigraph` plugin and will output all
graphs in a single run.

```
[haproxyng*]
env.socket /path/to/socket
env.clean prefix-
```

`haproxyng` takes two environmental options:

  * `socket` which is the path to the UNIX socket to HAProxy. This script only
    needs the `user` level permission to get read access to the statistics. No
    write or admin level access is required.
  * `clean` which can be used to strip common parts of a name from the
    configuration. For example if you're configuration is automatically
    generated and everything is prefixed with "staging-" or "production_" then
    put that (or any other regex) into `clean` and it will be cleaned from any
    titles before being output to Munin.

Beyond that, copy/symlink it to the `plugins/` directory on the relevant node
and wait for it to run. Running

    munin-run haproxyng config

is also possible to verify that it can see everything and output the
configuration data for Munin.

## Install instructions for debian

Go somewhere and get yourself the haproxyng script

```
cd /tmp/
wget -O haproxyng https://raw.githubusercontent.com/thenovacreator/monitoring-munin-haproxy/master/haproxyng  
```

Copy the script to munins plugin location, create a symlink and set the execution permission
```
mv haproxyng /usr/share/munin/plugins/haproxyng
ln -s /usr/share/munin/plugins/haproxyng /etc/munin/plugins/haproxyng
chmod 755 /usr/share/munin/plugins/haproxyng
```
Install the libswitch-perl package that is needed by the script
```
sudo apt-get install libswitch-perl
```
Define a socket for the plugin to get the data if not already done so. First you have to open the haproxy config file
```
nano /etc/haproxy/haproxy.cfg 
```
And put the socket definition into the global section
```
global
  [other config stuff ...]

  stats socket /run/haproxy-user.sock mode 666 level user

  [other config stuff ...]
```
As you can see the access rights for this is `666`. This can be a security issue. This is because haproxy creates its files under /run/ with root user and group. So for the munin user to use the socket there must be set the `666` permissions on the socket. After starting haproxy you could change the sockets user group as you want and can use a stronger persmissions set. You could use `chown root:munin /run/haproxy/user.sock` to do that. This is up to you. But keep in mind. As the server restarts the sockets are recreated and in this case could have the wrong permissions.

Now we have to setup munin to use the plugin. Therefor open the munin-node config file
```
nano /etc/munin/plugin-conf.d/munin-node
```
And add the haproxyng section with the right socket
```
[haproxyng*]
env.socket /run/haproxy-user.sock
```
Now restart the munin and haproxy services
```
service haproxy restart
service munin-node restart
```
If you want to test if everything is going right executed this command
```
munin-run haproxy
```
You should be the statistics to be printed

If you do not get errors and the plugin is not working you may take a look the munin log file
```
cat /var/log/munin-node.log
```

## Licence

This program is free software; you can redistribute it and/or
modify it under the terms of the GNU General Public License
as published by the Free Software Foundation; either version 2
of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
