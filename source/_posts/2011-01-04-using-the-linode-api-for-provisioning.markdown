---
layout: post
title: "Using the Linode API for provisioning"
date: 2011-01-04 16:48
comments: true
categories: 
---

<a href="http://www.linode.com" title="Linode.com" target="_blank">Linode</a> is a great VPS/Cloud provider and we use it to run a development/testing machine and a production MongoDB server. It also has an easy to use REST <a href="http://www.linode.com/api/" title="Linode API" target="_blank">API</a> which, unlike Amazon EC2, requires only an access key and no complicated X509 configuration. There is a nice Ruby library for this API on github: <https://github.com/rick/linode>

We use Puppet to manage node configuration once a node is provisioned (and this is cloud-provider agnostic), however provisioning the node requires use of the particular cloud provider's API and this is where the library above comes in. There are libraries like <a href="http://incubator.apache.org/libcloud/" title="libcloud" target="_blank">libcloud</a> that provide an abstraction layer that works with many cloud providers - using the Linode API directly made the most sense for us.

The Ruby Linode library above exposes the Linode API directly, and it is fairly low level (no model of the node, config, etc.). Since (re)provisioning requires successive interactions with the same Linode instance, as well as a notion of a static configuration which can be applied to the node, it's nice to have a higher level model of the virtual machine. We created a small library which wraps the Linode API and adds a couple of abstractions. You can find it here on GitHub:

<https://github.com/incandescent/linode-utils>

This linode-utils gem provides a simple model of a Linode Machine, as well as a small Linode configuration DSL.

Machine operations:

* boot
* reload
* shutdown
* get_disks
* delete_disks
* delete_non_swap_disks
* delete_configs
* used_disk_space
* create_disk_from_stackscript
* create_config
* wait_for_job

Here is how we (re)provision a Linode instance:

``` ruby
api = Linode.new(:api_key => api_key, :username => username, :password => password)

LinodeUtils::LOG.info "Looking for linode named '#{name}'"

LINODE = Machine.new(api, name)

LinodeUtils::LOG.info "Current linode status: " + LINODE.linode.status.to_s

if LINODE.linode.status != Linode::TERMINATED
  LINODE.shutdown
end

LinodeUtils::LOG.info "Deleting non-swap disks"
LINODE.delete_non_swap_disks

LinodeUtils::LOG.info "Deleting configs"
LINODE.delete_configs

root_disk_id = LINODE.create_disk_from_stackscript({
  :size => LinodeUtils::MAXIMUM_DISK_SIZE,
  :distro => DISTRO_LABEL,
  :stackscriptid => STACK_SCRIPT_ID,
  # stack script options
  :root_pub_key => pub_key,
  :fqdn => hostname
})

config_id = LINODE.create_config(KERNEL_LABEL, root_disk_id)
LinodeUtils::LOG.info "Booting Linode..."

LINODE.boot(config_id)
```

(this is the interesting bit; the full script is about twice as large but the rest is mostly setup and argument parsing)

There is also a small DSL for constructing the Linode "config". Here is an example:

``` ruby
config = LinodeUtils::LinodeConfig.build LinodeUtils.init_api do
  label "My favorite config"
  kernel /Latest 2\.6 Paravirt(?!.*x86_64.*)/
  comment "first comment"
  disk 0, :root_device => true
  comment "second comment"
  disk 1
end
```

After implementing this automation for Linode, I think we could probably easily move to another service. The basic steps are the same:</p>

1. provision an instance - this requires selecting or building an OS image, and ensuring a root SSH key is installed
1. install a first-boot script to perform config/bootstrapping (how this is implemented differs to some degree from service to service)
1. ssh as root to execute any commands required to finish bootstrapping (if you have a configuration server that is known to the bootstrapped node then this step can be eliminated)

On that note, stay tuned for future posts on bootstrapping a node with Puppet+Git, how to use Hudson and a "stable branch" Git methodology for continuous integration with Rails, and our foray into CouchDB-based desktop apps.


