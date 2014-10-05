Hiera-based Foreman ENC
=======================

This project use the `enc` script to combine Foreman and Hiera configurations.
Using Puppet Master ENC feature to provide YAML configurations to client.
The script merges Hiera configurations over Foreman without overwriting them.

The idea is for Foreman to define the scope for Hiear and allow optional or
temporary overwrite of "default" Hiera configurations.

The script uses Foreman [`external_node_v2.rb`][1] to get client configuration.
Then it uses the `environment` key to define the lookup scope in Hiera. With
the `hiera.lookup` function it "fills" the missing configuration or in other
words Foreman overwrites Hiera. The script will lookup for `parameters` and
`classes` in Hiera with the same syntax and structure as in Foreman.

Usage
--------
To use the ENC feature in Puppet master add the following lines to
`puppet.conf`:

    [master]
        node_terminus = exec
        external_nodes = /etc/puppet/enc

Make sure to download [external_node_v2.rb][1] and configure
`/etc/puppet/foreman.yaml`:

	:url: "http://foreman:3000"
	:puppetdir: "/var/lib/puppet"
	:facts: true
	:storeconfigs: true
	:timeout: 3

Environment configuration may be made in
`/etc/puppet/hiera/hiera_<environment>/global.yaml`

Check the data returned for a given node by executing `./enc <fqdn>`, just as
Puppet itself will do. Note that the `DEBUG` lines are printed to STDERR and
are not parsed by Puppet. Other option it to run `puppet agent -t` on the client
and check the full result on Puppet Master at
`/var/lib/puppet/yaml/node/<fqdn>.yaml`

Customization
-------------------
Edit `enc` and `hiera.yaml` to define Hiera hierarchy and yaml location.

Example
-----------

`/etc/puppet/hiera/hiera_production/global.yaml`:

    ---
    class:
      ntp:
        servers: - clock.example.com iburst
<!-- -->
`/etc/puppet/external_node_v2.rb hostname.example.com`:

	---
	environment: production
	classes:
	  ntp:
	    service_enable: true
	parameters:
	...

When the enc is queried for `hostname.example.com`, the result is the merged
hash of these values:

    $ ./enc hostname.example.com
    DEBUG: Sun Oct 05 11:34:10 -0400 2014: Hiera YAML backend starting
    DEBUG: Sun Oct 05 11:34:10 -0400 2014: Looking up classes in YAML backend
    DEBUG: Sun Oct 05 11:34:10 -0400 2014: Looking for data source global
    DEBUG: Sun Oct 05 11:34:10 -0400 2014: Found classes in global
    DEBUG: Sun Oct 05 11:34:10 -0400 2014: Looking up parameters in YAML backend
    DEBUG: Sun Oct 05 11:34:10 -0400 2014: Looking for data source global
    ---
    classes:
      ntp:
        service_enable: true
        servers:
	- clock.example.com iburst

[1]: https://github.com/theforeman/puppet-foreman/blob/master/files/external_node_v2.rb

