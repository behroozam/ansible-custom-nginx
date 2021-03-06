# custom_nginx

This role is building and installing nginx directly out of the source code.

It's providing an easy way to patch the nginx sources according to your needs.

The role is not made for the configuration of the nginx service itself. There
are already powerful roles out there. If you're looking for one I'd highly
recommend [jdauphant.nginx](https://github.com/jdauphant/ansible-role-nginx).

## Requirements

This role requires
[Ansible 2.6.0](https://docs.ansible.com/ansible/devel/roadmap/ROADMAP_2_6.html)
or higher in order to apply [patches](#2-apply-patches-to-the-source).

You can simply use pip to install (and define) a stable version:

```sh
pip install ansible==2.7.7
```

All platform requirements are listed in the metadata file.

## Install

```sh
ansible-galaxy install behroozam.custom_nginx
```

## Role Variables

The variables that can be passed to this role. You can find a brief description
in this paragraph. For all variables, take a look at
[defaults/main.yml](defaults/main.yml).

```yaml
# Version definition:
# Type: Int
custom_nginx_version: 1.15.3

# Basic settings for building nginx from sources.
# Align them with your build options.

# Paths:
# Type: Str
custom_nginx_sbin_path: /usr/local/nginx/sbin/nginx
custom_nginx_conf_path: /usr/local/nginx/conf/nginx.conf
custom_nginx_pid_path: /var/run/nginx.pid
custom_nginx_lock_path: /var/lock/nginx.lock

# Directories:
# Type: Str
custom_nginx_prefix_directory: /usr/local/nginx
custom_nginx_log_directory: /var/log/nginx
custom_nginx_cache_directory: /var/cache/nginx
custom_nginx_modules_directory: /usr/lib/nginx/modules
custom_nginx_static_modules_directory: /usr/local/src/modules

# In this section you can apply custom patches to nginx.
# You can find one example below in this document, see 2.1)
# Patches explained.
# Type: Dict
custom_nginx_patches:
  disable_h2c_table_update:
    dest_file: src/http/v2/ngx_http_v2.c
    patch_file: "roles/loadbalancer/files/nginx-patches/disable_h2c_table_update-{{ custom_nginx_version }}.patch"
    state: present

# In this section you can apply custom patches to nginx basedir.
# You can find one example below in this document, see 2.1)
# Patches explained.
# Type: Dict

custom_nginx_patches_dir:
  nginx_upstream_check_module_patch:
    dest_file: ""
    patch_file: roles/loadbalancer/files/nginx-patches/check_1.14.0+.patch
    state: present
    
# In this section you can download custom modules and compile them with nginx
# Type: Dict  
custom_nginx_modules:
  install_nginx_module_vts:
    repo: https://github.com/vozlt/nginx-module-vts.git
    version: master #default value is master 


# Build options
# Type: List
custom_nginx_build_options:
  - "--prefix={{ custom_nginx_prefix_directory }}"
  - "--sbin-path={{ custom_nginx_sbin_path }}"
  - "--modules-path={{ custom_nginx_modules_directory }}"
  - "--conf-path={{ custom_nginx_conf_path }}"
  - "--pid-path={{ custom_nginx_pid_path }}"
  - "--lock-path={{ custom_nginx_lock_path }}"
  - "--error-log-path={{ custom_nginx_log_directory }}/error.log"
  - "--http-log-path={{ custom_nginx_log_directory }}/access.log"
  - "--http-client-body-temp-path={{ custom_nginx_cache_directory }}/client_temp"
  - "--http-proxy-temp-path={{ custom_nginx_cache_directory }}/proxy_temp"
  - "--http-fastcgi-temp-path={{ custom_nginx_cache_directory }}/fastcgi_temp"
  - "--http-uwsgi-temp-path={{ custom_nginx_cache_directory }}/uwsgi_temp"
  - "--http-scgi-temp-path={{ custom_nginx_cache_directory }}/scgi_temp"
  - "--user={{ custom_nginx_user }}"
  - "--group={{ custom_nginx_group }}"
  - "--with-compat"
  - "--with-file-aio"
  - "--with-threads"
  - "--with-http_addition_module"
  - "--with-http_auth_request_module"
  - "--with-http_dav_module"
  - "--with-http_flv_module"
  - "--with-http_gunzip_module"
  - "--with-http_geoip_module=dynamic"
  - "--with-http_gzip_static_module"
  - "--with-http_mp4_module"
  - "--with-http_random_index_module"
  - "--with-http_realip_module"
  - "--with-http_secure_link_module"
  - "--with-http_slice_module"
  - "--with-http_ssl_module"
  - "--with-http_stub_status_module"
  - "--with-http_sub_module"
  - "--with-http_v2_module"
  - "--with-stream"
  - "--with-stream_geoip_module=dynamic"
  - "--with-stream_realip_module"
  - "--with-stream_ssl_module"
  - "--with-stream_ssl_preread_module"
  - "--add-module={{custom_nginx_static_modules_directory}}/nginx-module-vts"
```

## Examples

To keep the document lean the compile options are stripped.
You can find the nginx build options either in [this
document](#nginx-build-options) or online in the official [nginx
documentation](http://nginx.org/en/docs/configure.html).

### 1) Build nginx according to your needs

```yaml
- hosts: nginx
  vars:
    custom_nginx_version: 1.15.3
    custom_nginx_conf_path: /etc/nginx/nginx.conf
    custom_nginx_prefix_directory: /etc/nginx
    custom_nginx_sbin_path: /usr/sbin/nginx
    custom_nginx_build_options:
      - "--prefix={{ custom_nginx_prefix_directory }}"
      - "--sbin-path={{ custom_nginx_sbin_path }}"
      - "--modules-path={{ custom_nginx_modules_directory }}"
      - ...
  roles:
    - timorunge.custom_nginx
```

### 2) Apply patches to the source

```yaml
- hosts: nginx
  vars:
    custom_nginx_version: 1.15.3
    custom_nginx_patches:
      disable_h2c_table_update:
        dest_file: src/http/v2/ngx_http_v2.c
        patch_file: "disable_h2c_table_update-{{ custom_nginx_version }}.patch"
        state: present
    custom_nginx_build_options:
      - "--prefix={{ custom_nginx_prefix_directory }}"
      - "--sbin-path={{ custom_nginx_sbin_path }}"
      - "--modules-path={{ custom_nginx_modules_directory }}"
      - ...
  roles:
    - timorunge.custom_nginx
```

#### 2.1) Patches (a little bit) explained

```yaml
custom_nginx_patches:
  disable_h2c_table_update:
    # The destination file which should get the patch:
    dest_file: src/http/v2/ngx_http_v2.c
    # The file, locally in your file system, which contains the patch.
    # If you're calling the custom_nginx_patches from another role which is
    # called e.g. loadbalancer and you've the file stored inside this role
    # it should look like this:
    patch_file: "roles/loadbalancer/files/nginx-patches/disable_h2c_table_update-{{ custom_nginx_version }}.patch"
    # State of the patch, can be "present" (default) or "absent":
    state: present
```

### 3) Override init.d and systemd templates

```yaml
- hosts: nginx
  vars:
    custom_nginx_version: 1.15.3
    custom_nginx_init_template: roles/loadbalancer/templates/nginx.service.j2
    custom_nginx_service_template: roles/loadbalancer/templates/nginx.init.j2
    custom_nginx_build_options:
      - "--prefix={{ custom_nginx_prefix_directory }}"
      - "--sbin-path={{ custom_nginx_sbin_path }}"
      - "--modules-path={{ custom_nginx_modules_directory }}"
      - ...
  roles:
    - timorunge.custom_nginx
```

### 4) nginx configuration with jdauphant.nginx

Like mentioned in the beginning, this role is not build for the configuration
of nginx itself.

Here you have some example in order to get everything up and running together
with [jdauphant.nginx](https://github.com/jdauphant/ansible-role-nginx).

```yaml
- hosts: nginx
  vars:
    custom_nginx_version: 1.15.3
    custom_nginx_user: nginx
    custom_nginx_conf_path: /etc/nginx/nginx.conf
    custom_nginx_pid_path: /var/run/nginx.pid
    custom_nginx_sbin_path: /usr/sbin/nginx
    custom_nginx_prefix_directory: /etc/nginx
    custom_nginx_log_directory: /var/log/nginx
    custom_nginx_build_options:
      - "--prefix={{ custom_nginx_prefix_directory }}"
      - "--sbin-path={{ custom_nginx_sbin_path }}"
      - "--modules-path={{ custom_nginx_modules_directory }}"
      - ...
    # jdauphant.nginx
    nginx_installation_type: configuration-only
    nginx_installation_types_using_service: configuration-only
    nginx_binary_name: "{{ custom_nginx_sbin_path }}"
    nginx_conf_dir: "{{ custom_nginx_prefix_directory }}"
    nginx_log_dir: "{{ custom_nginx_log_directory }}"
    nginx_pid_file: "{{ custom_nginx_pid_path }}"
    nginx_service_name: nginx
    nginx_user: "{{ custom_nginx_user }}"
    nginx_http_params:
      - sendfile on
      - server_tokens off
      - ...
      - "access_log {{ custom_nginx_log_directory }}/access.log"
      - "error_log {{custom_nginx_log_directory}}/error.log {{ nginx_error_log_level }}"
    nginx_sites:
       foo:
         - listen 8080
         - server_name localhost
         - root /tmp/site1
         - location / { try_files $uri $uri/ /index.html; }
         - location /images/ { try_files $uri $uri/ /index.html; }
       bar:
         - listen 9090
         - server_name ansible
         - root /tmp/site2
         - location / { try_files $uri $uri/ /index.html; }
         - location /images/ { try_files $uri $uri/ /index.html; }
    nginx_configs:
       proxy:
          - proxy_set_header X-Real-IP $remote_addr
          - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
  roles:
    - timorunge.custom_nginx
    - jdauphant.nginx
```

### 5) Adding logrotate support

Per default there is no log rotation for the nginx logs since there is already
a powerful role out there. Take a look at
[nickhammond.logrotate](https://github.com/nickhammond/ansible-logrotate).

```yaml
- hosts: nginx
  vars:
    custom_nginx_version: 1.15.3
    custom_nginx_user: nginx
    custom_nginx_group: nginx
    custom_nginx_pid_path: /var/run/nginx.pid
    custom_nginx_log_directory: /var/log/nginx
    custom_nginx_build_options:
      - "--prefix={{ custom_nginx_prefix_directory }}"
      - "--sbin-path={{ custom_nginx_sbin_path }}"
      - "--modules-path={{ custom_nginx_modules_directory }}"
      - ...
      - "--error-log-path={{ custom_nginx_log_directory }}/error.log"
      - "--http-log-path={{ custom_nginx_log_directory }}/access.log"
      - ...
    # nickhammond.logrotate
    logrotate_scripts:
      - name: nginx
        paths:
          - "{{ custom_nginx_log_directory }}/*.log"
        options:
          - daily
          - missingok
          - rotate 14
          - compress
          - delaycompress
          - notifempty
          - "create 640 {{ custom_nginx_user }} {{ custom_nginx_group }}"
          - sharedscripts
        scripts:
          postrotate: "if [ -f {{ custom_nginx_pid_path }} ]; then kill -USR1 `cat {{ custom_nginx_pid_path }}` ; fi"
  roles:
    - timorunge.custom_nginx
    - nickhammond.logrotate
```

## nginx build options

An overview of the build options for nginx (1.15.3).

```sh
  --help                             print this message

  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
  --error-log-path=PATH              set error log pathname
  --pid-path=PATH                    set nginx.pid pathname
  --lock-path=PATH                   set nginx.lock pathname

  --user=USER                        set non-privileged user for
                                     worker processes
  --group=GROUP                      set non-privileged group for
                                     worker processes

  --build=NAME                       set build name
  --builddir=DIR                     set build directory

  --with-select_module               enable select module
  --without-select_module            disable select module
  --with-poll_module                 enable poll module
  --without-poll_module              disable poll module

  --with-threads                     enable thread pool support

  --with-file-aio                    enable file AIO support

  --with-http_ssl_module             enable ngx_http_ssl_module
  --with-http_v2_module              enable ngx_http_v2_module
  --with-http_realip_module          enable ngx_http_realip_module
  --with-http_addition_module        enable ngx_http_addition_module
  --with-http_xslt_module            enable ngx_http_xslt_module
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module    enable ngx_http_image_filter_module
  --with-http_image_filter_module=dynamic
                                     enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module           enable ngx_http_geoip_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_sub_module             enable ngx_http_sub_module
  --with-http_dav_module             enable ngx_http_dav_module
  --with-http_flv_module             enable ngx_http_flv_module
  --with-http_mp4_module             enable ngx_http_mp4_module
  --with-http_gunzip_module          enable ngx_http_gunzip_module
  --with-http_gzip_static_module     enable ngx_http_gzip_static_module
  --with-http_auth_request_module    enable ngx_http_auth_request_module
  --with-http_random_index_module    enable ngx_http_random_index_module
  --with-http_secure_link_module     enable ngx_http_secure_link_module
  --with-http_degradation_module     enable ngx_http_degradation_module
  --with-http_slice_module           enable ngx_http_slice_module
  --with-http_stub_status_module     enable ngx_http_stub_status_module

  --without-http_charset_module      disable ngx_http_charset_module
  --without-http_gzip_module         disable ngx_http_gzip_module
  --without-http_ssi_module          disable ngx_http_ssi_module
  --without-http_userid_module       disable ngx_http_userid_module
  --without-http_access_module       disable ngx_http_access_module
  --without-http_auth_basic_module   disable ngx_http_auth_basic_module
  --without-http_mirror_module       disable ngx_http_mirror_module
  --without-http_autoindex_module    disable ngx_http_autoindex_module
  --without-http_geo_module          disable ngx_http_geo_module
  --without-http_map_module          disable ngx_http_map_module
  --without-http_split_clients_module disable ngx_http_split_clients_module
  --without-http_referer_module      disable ngx_http_referer_module
  --without-http_rewrite_module      disable ngx_http_rewrite_module
  --without-http_proxy_module        disable ngx_http_proxy_module
  --without-http_fastcgi_module      disable ngx_http_fastcgi_module
  --without-http_uwsgi_module        disable ngx_http_uwsgi_module
  --without-http_scgi_module         disable ngx_http_scgi_module
  --without-http_grpc_module         disable ngx_http_grpc_module
  --without-http_memcached_module    disable ngx_http_memcached_module
  --without-http_limit_conn_module   disable ngx_http_limit_conn_module
  --without-http_limit_req_module    disable ngx_http_limit_req_module
  --without-http_empty_gif_module    disable ngx_http_empty_gif_module
  --without-http_browser_module      disable ngx_http_browser_module
  --without-http_upstream_hash_module
                                     disable ngx_http_upstream_hash_module
  --without-http_upstream_ip_hash_module
                                     disable ngx_http_upstream_ip_hash_module
  --without-http_upstream_least_conn_module
                                     disable ngx_http_upstream_least_conn_module
  --without-http_upstream_random_module
                                     disable ngx_http_upstream_random_module
  --without-http_upstream_keepalive_module
                                     disable ngx_http_upstream_keepalive_module
  --without-http_upstream_zone_module
                                     disable ngx_http_upstream_zone_module

  --with-http_perl_module            enable ngx_http_perl_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-perl_modules_path=PATH      set Perl modules path
  --with-perl=PATH                   set perl binary pathname

  --http-log-path=PATH               set http access log pathname
  --http-client-body-temp-path=PATH  set path to store
                                     http client request body temporary files
  --http-proxy-temp-path=PATH        set path to store
                                     http proxy temporary files
  --http-fastcgi-temp-path=PATH      set path to store
                                     http fastcgi temporary files
  --http-uwsgi-temp-path=PATH        set path to store
                                     http uwsgi temporary files
  --http-scgi-temp-path=PATH         set path to store
                                     http scgi temporary files

  --without-http                     disable HTTP server
  --without-http-cache               disable HTTP cache

  --with-mail                        enable POP3/IMAP4/SMTP proxy module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-mail_ssl_module             enable ngx_mail_ssl_module
  --without-mail_pop3_module         disable ngx_mail_pop3_module
  --without-mail_imap_module         disable ngx_mail_imap_module
  --without-mail_smtp_module         disable ngx_mail_smtp_module

  --with-stream                      enable TCP/UDP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
  --with-stream_ssl_module           enable ngx_stream_ssl_module
  --with-stream_realip_module        enable ngx_stream_realip_module
  --with-stream_geoip_module         enable ngx_stream_geoip_module
  --with-stream_geoip_module=dynamic enable dynamic ngx_stream_geoip_module
  --with-stream_ssl_preread_module   enable ngx_stream_ssl_preread_module
  --without-stream_limit_conn_module disable ngx_stream_limit_conn_module
  --without-stream_access_module     disable ngx_stream_access_module
  --without-stream_geo_module        disable ngx_stream_geo_module
  --without-stream_map_module        disable ngx_stream_map_module
  --without-stream_split_clients_module
                                     disable ngx_stream_split_clients_module
  --without-stream_return_module     disable ngx_stream_return_module
  --without-stream_upstream_hash_module
                                     disable ngx_stream_upstream_hash_module
  --without-stream_upstream_least_conn_module
                                     disable ngx_stream_upstream_least_conn_module
  --without-stream_upstream_random_module
                                     disable ngx_stream_upstream_random_module
  --without-stream_upstream_zone_module
                                     disable ngx_stream_upstream_zone_module

  --with-google_perftools_module     enable ngx_google_perftools_module
  --with-cpp_test_module             enable ngx_cpp_test_module

  --add-module=PATH                  enable external module
  --add-dynamic-module=PATH          enable dynamic external module

  --with-compat                      dynamic modules compatibility

  --with-cc=PATH                     set C compiler pathname
  --with-cpp=PATH                    set C preprocessor pathname
  --with-cc-opt=OPTIONS              set additional C compiler options
  --with-ld-opt=OPTIONS              set additional linker options
  --with-cpu-opt=CPU                 build for the specified CPU, valid values:
                                     pentium, pentiumpro, pentium3, pentium4,
                                     athlon, opteron, sparc32, sparc64, ppc64

  --without-pcre                     disable PCRE library usage
  --with-pcre                        force PCRE library usage
  --with-pcre=DIR                    set path to PCRE library sources
  --with-pcre-opt=OPTIONS            set additional build options for PCRE
  --with-pcre-jit                    build PCRE with JIT compilation support

  --with-zlib=DIR                    set path to zlib library sources
  --with-zlib-opt=OPTIONS            set additional build options for zlib
  --with-zlib-asm=CPU                use zlib assembler sources optimized
                                     for the specified CPU, valid values:
                                     pentium, pentiumpro

  --with-libatomic                   force libatomic_ops library usage
  --with-libatomic=DIR               set path to libatomic_ops library sources

  --with-openssl=DIR                 set path to OpenSSL library sources
  --with-openssl-opt=OPTIONS         set additional build options for OpenSSL

  --with-debug                       enable debug logging
```

## Testing

[![Build Status](https://travis-ci.org/timorunge/ansible-custom-nginx.svg?branch=master)](https://travis-ci.org/timorunge/ansible-custom-nginx)

Tests are done with [Docker](https://www.docker.com) and
[docker_test_runner](https://github.com/timorunge/docker-test-runner) which
brings up the following containers:

- CentOS 7
- Debian 8.10 (Jessie)
- Debian 9.4 (Stretch)
- Debian 10 (Buster)
- Ubuntu 16.04 (Xenial Xerus)
- Ubuntu 18.04 (Bionic Beaver)
- Ubuntu 18.10 (Cosmic Cuttlefish)

Ansible 2.7.7 is installed on all containers and a
[test playbook](tests/test.yml) is getting applied.

For further details and additional checks take a look at the
[docker_test_runner configuration](tests/docker_test_runner.yml) and the
[Docker entrypoint](tests/docker/docker-entrypoint.sh).

```sh
# Testing locally:
curl https://raw.githubusercontent.com/timorunge/docker-test-runner/master/install.sh | sh
./docker_test_runner.py -f tests/docker_test_runner.yml
```

Since the build time on Travis is limited for public repositories the
[automated tests](tests/docker_test_runner.travis.yml) are limited to:

- CentOS 7
- Debian 9.4 (Stretch)
- Ubuntu 16.04 (Xenial Xerus)
- Ubuntu 18.04 (Bionic Beaver)

## Dependencies

None

## License

[BSD 3-Clause "New" or "Revised" License](LICENSE)

## Authors Information

- Timo Runge
- Behrooz Hasanbeygi