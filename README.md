## nginx

[![Build Status](https://travis-ci.org/Oefenweb/ansible-nginx.svg?branch=master)](https://travis-ci.org/Oefenweb/ansible-nginx) [![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-nginx-blue.svg)](https://galaxy.ansible.com/Oefenweb/nginx)

Set up (the latest version of) [NGINX](http://nginx.org/) in Debian-like systems.

#### Requirements

* `python-apt`

#### Variables

* `nginx_version`: [default: `stable`]: Version to install (e.g. `development`)

* `nginx_dependencies`: [default: `['nginx']`]: Packages to install
* `nginx_install`: [default: `[]`]: Additional packages to install

* `nginx_core_directives`: [default: `["user {{ nginx_preset_user }} {{ nginx_preset_group }}", "worker_processes {{ nginx_preset_processes }}", "pid {{ nginx_preset_pid }}"]`]: 
* `nginx_events_directives`: [default: `["worker_connections {{ nginx_preset_worker_connections }}"]`]: 
* `nginx_http_directives`: [default: `[]`]: 
* `nginx_stream_directives`: [optional]: 
* `nginx_mail_directives`: [optional]: 

* `nginx_present_paths`: [default: `[]`]: Directories to be created
* `nginx_absent_paths`: [default: `[]`]: Paths to be removed

* `nginx_conf_d_include_files`: [default: `[]`]: 
* `nginx_snippets_include_files`: [default: `[]`]: 
* `nginx_sites_available_include_files`: [default: `[]`]: 
* `nginx_streams_available_include_files`: [default: `[]`]: 
* `nginx_mails_available_include_files`: [default: `[]`]: 

* `nginx_ssl_map`: [default: `[]`]: SSL declarations
* `nginx_ssl_map.{n}.src`: The local path of the file to copy, can be absolute or relative (e.g. `../../../files/nginx/etc/nginx/ssl/star-example-com.pem`)
* `nginx_ssl_map.{n}.dest`: The remote path of the file to copy (e.g. `/etc/nginx/ssl/star-example-com.pem`)
* `nginx_ssl_map.{n}.owner`: The name of the user that should own the file (optional, default `root`)
* `nginx_ssl_map.{n}.group`: The name of the group that should own the file (optional, default `root`)
* `nginx_ssl_map.{n}.mode`: The mode of the file, such as 0644 (optional, default `0640`)

## Dependencies

None

#### Examples

##### 1

```yaml
---
- hosts: all
  roles:
    - nginx
``

##### 2

```yaml
---
- hosts: all
  roles:
    - nginx
``

#### License

MIT

#### Author Information

Mischa ter Smitten (based on work of [jdauphant](https://github.com/jdauphant) and [geerlingguy](https://github.com/geerlingguy))

#### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/Oefenweb/ansible-nginx/issues)!
