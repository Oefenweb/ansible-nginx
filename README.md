## nginx

[![CI](https://github.com/Oefenweb/ansible-nginx/workflows/CI/badge.svg)](https://github.com/Oefenweb/ansible-nginx/actions?query=workflow%3ACI)
[![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-nginx-blue.svg)](https://galaxy.ansible.com/Oefenweb/nginx)

Set up (the latest version of) [NGINX](http://nginx.org/) in Debian-like systems.

#### Requirements

* `software-properties-common` (will be installed)
* `dirmngr` (will be installed)

#### Variables

* `nginx_use_ppa`: [default: `true`]: Whether or not to add the PPA (for installation)

* `nginx_version`: [default: `stable`]: Version to install (e.g. `development`)

* `nginx_dependencies`: [default: `['nginx']`]: Packages to install
* `nginx_install`: [default: `[]`]: Additional packages to install

* `nginx_core_directives`: [default: `["user {{ nginx_preset_user }} {{ nginx_preset_group }}", "worker_processes {{ nginx_preset_processes }}", "pid {{ nginx_preset_pid }}"]`]: Core functionality directives ([see](https://nginx.org/en/docs/ngx_core_module.html#directives))
* `nginx_events_directives`: [default: `["worker_connections {{ nginx_preset_worker_connections }}"]`]: Events functionality directives ([see](http://nginx.org/en/docs/ngx_core_module.html#events))
* `nginx_http_directives`: [default: `[]`]: HTTP functionality directives ([see](https://nginx.org/en/docs/http/ngx_http_core_module.html))
* `nginx_stream_directives`: [optional]: Stream functionality directives ([see](https://nginx.org/en/docs/stream/ngx_stream_core_module.html))
* `nginx_mail_directives`: [optional]: Mail functionality directives ([see](https://nginx.org/en/docs/mail/ngx_mail_core_module.html))

* `nginx_present_paths`: [default: `[]`]: Directories to be created
* `nginx_absent_paths`: [default: `[]`]: Paths to be removed

* `nginx_conf_d_include_files`: [default: `[]`]: `conf.d` file declarations (in `/etc/nginx/conf.d`)
* `nginx_conf_d_include_files.{n}.name`: [required]: The name of the file (e.g. `ssl.conf`)

* `nginx_snippets_include_files`: [default: `[]`]: `snippets` file declarations (in `/etc/nginx/snippets`)
* `nginx_snippets_include_files.{n}.name`: [required]: The name of the file (e.g. `fastcgi_param`)

* `nginx_sites_available_include_files`: [default: `[]`]: `sites_available` file declarations (in `/etc/nginx/sites_available`)
* `nginx_sites_available_include_files.{n}.name`: [required]: The name of the file (e.g. `default-80.conf`)
* `nginx_sites_available_include_files.{n}.state`: [default `enabled`]: The state of the file. Settings this to `enabled` will create a symlink in `sites_enabled`

* `nginx_streams_available_include_files`: [default: `[]`]: `streams_available` file declarations (in `/etc/nginx/streams_available`)
* `nginx_streams_available_include_files.{n}.name`: [required]: The name of the file (e.g. `dns.conf`)
* `nginx_streams_available_include_files.{n}.state`: [default `enabled`]: The state of the file. Settings this to `enabled` will create a symlink in `streams_enabled`

* `nginx_mails_available_include_files`: [default: `[]`]: `mails_available` file declarations (in `/etc/nginx/mails_available`)
* `nginx_mails_available_include_files.{n}.name`: [required]: The name of the file (e.g. `mail.conf`)
* `nginx_mails_available_include_files.{n}.state`: [default `enabled`]: The state of the file. Settings this to `enabled` will create a symlink in `mails_enabled`

* `nginx_ssl_map`: [default: `[]`]: SSL declarations
* `nginx_ssl_map.{n}.src`: The local path of the file to copy, can be absolute or relative (e.g. `../../../files/nginx/etc/nginx/ssl/star-example-com.pem`)
* `nginx_ssl_map.{n}.dest`: The remote path of the file to copy (e.g. `/etc/nginx/ssl/star-example-com.pem`)
* `nginx_ssl_map.{n}.owner`: The name of the user that should own the file (optional, default `root`)
* `nginx_ssl_map.{n}.group`: The name of the group that should own the file (optional, default `root`)
* `nginx_ssl_map.{n}.mode`: The mode of the file, such as 0644 (optional, default `0640`)

## Dependencies

None

#### Examples

##### Simple, single vhost on port 80

```yaml
---
- hosts: all
  roles:
    - nginx
  vars:
    nginx_http_directives:
      - |
        include {{ nginx_conf_path }}/mime.types;
        default_type application/octet-stream;

        include {{ nginx_conf_d.path }}/*.conf;
        include {{ nginx_sites_enabled.path }}/*.conf;

    nginx_sites_available_include_files:
      - name: default-80.conf
        directives:
          - |
            server {
              listen 80;
              server_name _ default-80 "";
              location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
              }
            }
```

##### Advanced, multiple vhosts, stream and mail

```yaml
---
- hosts: all
  roles:
    - nginx
  vars:
    nginx_http_directives:
      - |
        ##
        # Basic Settings
        ##
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include {{ nginx_conf_path }}/mime.types;
        default_type application/octet-stream;

        ##
        # Logging Settings
        ##
        access_log {{ nginx_var_log.path }}/access.log;
        error_log {{ nginx_var_log.path }}/error.log;

        ##
        # Virtual Host Configs
        ##
        include {{ nginx_conf_d.path }}/*.conf;
        include {{ nginx_sites_enabled.path }}/*.conf;

    # optional
    nginx_stream_directives:
      - "include {{ nginx_streams_enabled.path }}/*.conf"

    # optional
    nginx_mail_directives:
      - |
        server_name       mail.example.com;
        auth_http         localhost:9000/cgi-bin/nginxauth.cgi;

        imap_capabilities IMAP4rev1 UIDPLUS IDLE LITERAL+ QUOTA;

        pop3_auth         plain apop cram-md5;
        pop3_capabilities LAST TOP USER PIPELINING UIDL;

        smtp_auth         login plain cram-md5;
        smtp_capabilities "SIZE 10485760" ENHANCEDSTATUSCODES 8BITMIME DSN;
        xclient           off;

        include {{ nginx_mails_enabled.path }}/*.conf;

    nginx_conf_d_include_files:
      - "{{ nginx_preset_conf_d_ssl }}"
      - "{{ nginx_preset_conf_d_gzip }}"

    nginx_snippets_include_files:
      - "{{ nginx_preset_conf_d_fastcgi_param }}"
      - "{{ nginx_preset_conf_d_scgi_param }}"
      - "{{ nginx_preset_conf_d_uwsgi_param }}"

    nginx_sites_available_include_files:
      - name: default-80.conf
        directives:
          - |
            server {
              listen 80;
              server_name _ default-80 "";
              location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
              }
            }

      - name: default-81.conf
        directives:
          - |
            server {
              listen 81;
              server_name _ default-81 "";
              root /usr/share/nginx/html;
              location / { try_files $uri $uri/ /index.html; }
              location /images/ { try_files $uri $uri/ /index.html; }
            }
      - name: default-82.conf
        directives:
          - server { listen 82; server_name _ default-82 ""; root /usr/share/nginx/html; location / { root /usr/share/nginx/html; index index.html index.htm; } }
      - name: default-83.conf
        directives:
          - |
            server {
              listen 83;
              server_name _ default-83 "";
              root /usr/share/nginx/html;
              location / {
                root /usr/share/nginx/html;
                index index.html index.htm;
              }
            }

    nginx_streams_available_include_files:
      - name: dns.conf
        directives:
          - |
            upstream dns {
               server 8.8.8.8:53;
               server 8.8.4.4:53;
            }

            server {
              listen {{ ansible_lo['ipv4']['address'] }}:53 udp;
              proxy_responses 1;
              proxy_timeout 20s;
              proxy_pass dns;
            }

    nginx_mails_available_include_files:
      - name: mail.conf
        directives:
          - |
            server {
              listen   1025;
              protocol smtp;
            }
            server {
              listen   1110;
              protocol pop3;
              proxy_pass_error_message on;
            }
            server {
              listen   1143;
              protocol imap;
            }
            server {
              listen   1587;
              protocol smtp;
            }

    nginx_absent_paths:
      - "{{ nginx_conf_path }}/fastcgi_params"
      - "{{ nginx_conf_path }}/scgi_params"
      - "{{ nginx_conf_path }}/uwsgi_params"

      - "{{ nginx_conf_d.path }}/default.conf"
```

#### License

MIT

#### Author Information

Mischa ter Smitten (based on work of [jdauphant](https://github.com/jdauphant) and [geerlingguy](https://github.com/geerlingguy))

#### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/Oefenweb/ansible-nginx/issues)!
