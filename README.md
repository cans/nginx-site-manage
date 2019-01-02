nginx-site-manage
=================

A Role to manage nginx "sites" (virtual hosts configuration).

A lot of distributions allow for managing virtual hosts configuration
via symbolic links between a directory where configuration files are
stored, and another from which they are loaded. One can then easily
enable or disable a configuration by adding or removing as symlink to
the file in the former directory into the latter one. This is the kind
of setup this role deals with.


Describing the sites to install
"""""""""""""""""""""""""""""""

Each site is describe as follows:

```yaml
- site_template: "<path to a configuration file template>"
  site_name: "<name of the site>"
  site_state: "enabled"  # Either enabled or disabled
  # Add any additional key your template requires here.
  # They will be available to the template as `{{item.<key>}}`
```

This roles comes with a few basic templates, for simple use
cases, you can expand on. See the example playbooks below for more.

Note that for example, you can use templates from outside the role:

```yaml
- site_template: "{{play_dir}}/templates/nginx/my-template.j2"
  site_name: "my-site"
  site_state: "disabled"
  some_extra_variable: "this role does not care"
```

What this role doesn't do
"""""""""""""""""""""""""

This role does not install Nginx. It is not it's role.


Requirements
------------

This role has no requirements.


Role Variables
--------------

All variables in this role are namespaces with the prefix `nginxsites_`

Defaults
""""""""

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


Template variables
""""""""""""""""""

Here follows a description of the variables used by the _built-in_ templates.

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

The following will example configure a virtual host named `hello``
which proxies all request to a service running on the local host bound
to port 8080 (cf. ``. It will also make sure the configuration `default`

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
           # site remove only cares about the site name.
           # But you could have more variables defined here.
	   # Removing an existing site is a easy as cutting
	   # its description off of the `nginxsites_sites_absent` list
	   # and pasting it into the `nginxsites_sites_present` one.
```




License
-------

GPLv2


Author Information
------------------

Copyright © 2018-2019, Nicolas CANIART
