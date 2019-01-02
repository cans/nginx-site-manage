nginx-site-manage
=================

A Role to manage nginx "sites" (virtual hosts configuration).

A lot of distributions allow for managing virtual hosts configuration
via symbolic links between a directory where configuration files are
stored, and another from which they are loaded. One can then easily
enable or disable a configuration by adding or removing as symlink to
the file in the former directory into the latter one. This is the kind
of setup this role deals with.


### Describing the sites to install


Each site is describe as follows:

```yaml
- site_template: "<path to a configuration file template>"
  site_name: "<name of the site>"
  site_state: "enabled"  # Either enabled or disabled
  # Add any additional key/value praire your template requires
  # here. They will be available to the template as `{{item.<key>}}`
```

Note that the `site_name` will determine the name of the configuration
file. It should hence be valid file name.

This roles comes with a few basic templates, for simple use
cases, you can expand on. See the example playbooks below for more.

Note that, though a few templates are hereby provided, you can use
templates from outside this role. Here is one way to proceed:

```yaml
- site_template: "{{play_dir}}/templates/nginx/my-template.j2"
  site_name: "my-site"
  site_state: "disabled"
  some_extra_variable: "this role does not care"
```


### What this role doesn't do

This role does not install Nginx. It is not it's role.


Requirements
------------

This role has no requirements.


Role Variables
--------------

All variables in this role are namespaced with the prefix `nginxsites_`


### Defaults

- `nginxsites_config_dir`: nginx's configuration directory (default:
  `/etc/nginx`)
- `nginxsites_sites_available_dir`: directory from which nginx's loads
  virtual hosts configuration (default:
  `{{nginxsites_config_dir}}/sites-available`).
- `nginxsites_sites_enabled_dir`: directory in which store virtual hosts
  configuration (default: `{{nginxsites_config_dir}}/sites-available`).
- `nginxsites_sites_present`: a list of site names that need to be present
  in the `` directory. Each site is describe as explained above;
- `nginxsites_sites_absent`: a list of sites names that must not be
  available. Allows sites removal. (default: `[]` an empty list)


### Template variables

Here follows a description of the variables used by the _built-in_
[templates](./templates).

- `site_fqdn`: the fully qualified domain name as which 
  the site will be published (default: None must be specified)
  Applies to templates: `proxied-site.j2`
- `site_ip`: the IP address of the machine or container
  actually providing the service.
  Applies to templates: `proxied-site.j2`
- `site_port`: the TCP port the service is bound to.
  Applies to templates: `proxied-site.j2`



Dependencies
------------

This role has no dependency.


Example Playbook
----------------

The following will example configure a virtual host named `hello`
which proxies all request to a service running on the local host bound
to port 8080 (cf. the `nginxsites_sites_present` variable).

It will also make sure the `default` configuration is removed (_cf._
the `nginxsites_sites_absent` variable).

```yaml
- hosts: servers
  roles:
     - role: "cans.nginx-site-manage"
       nginxsites_sites_present:
         - site_name: hello
           site_template: "proxied-site.j2"  # Use built-in configuration template.
           site_state: "disabled"
           site_ip: "127.0.0.1"
           site_port: 8080
       nginxsites_sites_absent:
         - site_name: default
           # Site removal only cares about the `site_name'.
           # But you could have more variables defined here.
           # Removing an existing site is as easy as cutting
           # its description off of the `nginxsites_sites_present` list
           # and pasting it into the `nginxsites_sites_absent` one.
           # Or do the converse to restore a delete site.
```

The next example installs, at once, several sites defined in other
roles. Here we assume the variables `service1_nginxsites_sites` and
`service2_nginxsites_sites` are defined in the roles `service1` and
`service2` respectively. We assume as well those variables are defined
as explained above.

```yaml
---
- hosts: servers
  roles:

     - role: "service1"
     - role: "service2"
     
     - role: "cans.nginx-site-manage"
       nginxsites_sites_present: "{{ service1_nginxsites_sites + service2_nginxsites_sites }}"

```
Below is an example of how the `service1_nginxsites_sites` variable
could be defined in, e.g. `roles/service1/vars/main.yml`.
```yaml
---
service1_nginxsites_sites:
- site_name: "µservice1a"
  site_template: "proxied-site.j2"
  site_state: "disabled"
  site_ip: "127.0.0.1"
  site_port: 8080
- site_name: "µservice1b"
  # Use a specific template
  site_template: "roles/service1/templates/nginx/µservice2.j2"
  site_state: "disabled"
  # Define variables specific to the template.
  service1b_root: "/var/lib/service2/www"
  service1b_privatekey_file: "/etc/ssl/private/µservice2.key"
  service1b_certificate_file: "/etc/ssl/certs/µservice2.pem"
  # ...
```


License
-------

GPLv2


Author Information
------------------

Copyright © 2018-2019, Nicolas CANIART
