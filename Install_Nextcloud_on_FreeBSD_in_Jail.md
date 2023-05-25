# 在 FreeBSD 的 jail 中安装 NextCloud

 - 原文链接：<https://github.com/possnfiffer/bsd-pw/blob/gh-pages/docs/Install_Nextcloud_on_FreeBSD_in_Jail.md>
 - 作者：Roller Angel
 - 译者：飞鱼
 - 校对整理：ykla


### 在我的实验室里，我的 OPNSense 防火墙同时也是我的 DHCP 服务器，默认网关设置为 172.16.28.1，而且它还充当我的 DNS

在主机上运行：

```
vi /etc/sysctl.conf
```

添加内容：

```
# Allow jail raw sockets
security.jail.allow_raw_sockets=1

# Allow upgrades in jail
security.jail.chflags_allowed=1
```

运行以下命令：

```
sysctl security.jail.allow_raw_sockets=1
```

```
sysctl security.jail.chflags_allowed=1
```

```
vi /boot/loader.conf
```

添加内容：

```
# RACCT/RCTL Resource limits
kern.racct.enable=1
```

运行以下命令：

```
zfs create -o mountpoint=/jail zroot/jail
zfs create -o mountpoint=/jail/nextcloud zroot/jail/nextcloud
zfs create -o mountpoint=/jail/nextcloud/var/db/postgres/data -o recordsize=8k zroot/jail/nextcloud/pgsql
zfs get -r recordsize zroot/jail
```

```
fetch -o - http://ftp.freebsd.org/pub/FreeBSD/releases/amd64/12.0-RELEASE/base.txz | tar --unlink -xpJf - -C /jail/nextcloud
ls /jail/nextcloud
```

```
vi /etc/jail.conf
```

添加以下配置：

```
nextcloud {
  host.hostname = nextcloud.lab.bsd.pw;
  ip4.addr = 172.16.28.2;
  interface = em0;
  path = /jail/nextcloud;
  exec.start = "/bin/sh /etc/rc";
  exec.stop = "/bin/sh /etc/rc.shutdown";
  exec.clean;
  mount.devfs;
  allow.raw_sockets;
  sysvsem = new;
  sysvshm = new;
  sysvmsg = new;
}
```

运行以下命令：

```
sysrc jail_enable=YES
sysrc jail_nextcloud_mount_enable=YES
vi /etc/hosts
```

添加以下配置：

```
172.16.28.2 nextcloud.lab.bsd.pw nextcloud
```

运行以下命令：

```
service jail restart nextcloud
jexec 1 tcsh
vi /etc/hosts
```

添加以下配置：

```
172.16.28.2 nextcloud.lab.bsd.pw nextcloud
```

运行以下命令：

```
jexec 1 tcsh
newaliases -v
cp /usr/share/zoneinfo/America/Denver /etc/localtime
vi /etc/rc.conf
```

添加以下配置：

```
# DAEMONS | yes
syslogd_flags="-s -s"
sshd_enable=YES
# php_fpm_enable=YES
# postgresql_enable=YES
# postgresql_class=postgres
# postgresql_data=/var/db/postgres/data
# memcached_enable=YES
# memcached_flags="-l 172.16.28.2"
# nginx_enable=YES

# DAEMONS | no
sendmail_enable=NONE
sendmail_submit_enable=NO
sendmail_outbound_enable=NO
sendmail_msp_queue_enable=NO

# OTHER
clear_tmp_enable=YES
clear_tmp_X=YES
extra_netfs_types=NFS
dumpdev=NO
update_motd=NO
keyrate=fast
```

运行以下命令：

```
vi /etc/cron.d/sendmail-clean-clientmqueue
```

添加以下配置：

```
# CLEAN SENDMAIL
0 * * * * root /bin/rm -r -f /var/spool/clientmqueue/*
```

运行以下命令：

```
exit
service jail restart nextcloud
jls
jexec nextcloud tcsh
sockstat -l4
vi /etc/resolv.conf
```

添加以下配置：

```
nameserver 172.16.28.1
```

运行以下命令：

```
ping -c 3 bsd.pw
exit
```

## 使用 Poudriere 构建我们想要的 postgres

在主机上运行以下命令：

```
pkg install -y poudriere
vi nextcloudpkglist
```

添加以下配置：

```
www/nextcloud
www/nginx
databases/memcached
security/sudo
databases/postgresql10-server
www/php72-opcache
devel/php72-intl
mail/cclient
mail/php72-imap
math/php72-gmp
ftp/php72-ftp
```

运行以下命令：

```
vi /usr/local/etc/poudriere.d/amd64-12-0-make.conf
```

添加以下配置：

```
DEFAULT_VERSIONS += php=7.2
DEFAULT_VERSIONS += pgsql=10
OPTIONS_UNSET += MYSQL
OPTIONS_SET += PGSQL
```

运行以下命令：

```
poudriere bulk -j amd64-12-0 -p head -f nextcloudpkglist
```

### 当 jail 启动后，用 nullfs 将 Poudriere 软件包仓库挂载到 jail 内

运行以下命令：

```
service jail start nextcloud
jexec nextcloud tcsh
mkdir /mnt/amd64-12-0-head
exit
```

这个 fstab 的东西不起作用......以后会研究的，现在运行 fstab 之后的 `mount -t` 命令。

```
vi /etc/fstab.nextcloud
```

添加以下内容：

```
/usr/local/poudriere/data/packages/amd64-12-0-head    /mnt/amd64-12-0-head    nullfs    rw    0    0
```

使用 mount 命令来代替 fstab （这是一个手动命令，不会自动运行）。

```
mount -t nullfs /usr/local/poudriere/data/packages/amd64-12-0-head /jail/nextcloud/mnt/amd64-12-0-head
```

运行以下命令：

```
service jail restart nextcloud
jexec nextcloud tcsh
```

在 jail 中创建一个 pkg 仓库：

```
mkdir -p /usr/local/etc/pkg/repos
```

禁止 FreeBSD 中 pkg 包管理器的系统级源

```
vi /etc/pkg/FreeBSD.conf
```

将 `enabled` 选项改成 `no`：


```
FreeBSD: {
    enabled: no
}
```

创建一个替换 `FreeBSD.conf` 的源配置文件：

```
vi /usr/local/etc/pkg/repos/amd64-12-0.conf
```

添加以下配置：

```
amd64-12-0: {
    url: “file:///mnt/amd64-12-0-head”,
    enabled: yes,
}
```

用新的源来重新安装所有的包：

运行以下命令：

```
pkg upgrade -fy
pkg install -y www/nextcloud www/nginx databases/memcached security/sudo databases/postgresql10-server www/php72-opcache devel/php72-intl mail/cclient mail/php72-imap math/php72-gmp ftp/php72-ftp
```

## 升级 Poudriere

在主机上运行以下命令：

```
vi /usr/local/etc/poudriere.conf
```

```
CHECK_CHANGED_OPTIONS=verbose
CHECK_CHANGED_DEPS=yes
```

```
poudriere jail -j amd64-12-0 -u
```

```
poudriere ports -p head -u
poudriere bulk -j amd64-12-0 -p head -f nextcloudpkglist
```

从 jail 更新

```
jexec nextcloud tcsh
pkg update && pkg upgrade -y
```

## 设置 PostgreSQL

### 我确定我开启了 `poudriere options` 参数，并在 nextcloud 上启用了 memcached 支持，稍后我将把它添加到 `make.conf` 文件中。

在 jail 中运行以下命令：

```
vi /etc/login.conf
```

将以下设置添加到文件的末尾：

```
postgres:\
        :lang=en_US.UTF-8:\
        :setenv=LC_COLLATE=C:\
        :tc=default:
```

运行以下命令：

```
cap_mkdb /etc/login.conf
exit
service jail restart nextcloud
jexec nextcloud tcsh
chown postgres:postgres /var/db/postgres/data
vi /etc/rc.conf
```
：
更新配置：

```
# DAEMONS | yes
syslogd_flags="-s -s"
sshd_enable=YES
# php_fpm_enable=YES
postgresql_enable=YES
postgresql_class=postgres
postgresql_data=/var/db/postgres/data
# memcached_enable=YES
# memcached_flags="-l 172.16.28.2"
# nginx_enable=YES

# DAEMONS | no
sendmail_enable=NONE
sendmail_submit_enable=NO
sendmail_outbound_enable=NO
sendmail_msp_queue_enable=NO

# OTHER
clear_tmp_enable=YES
clear_tmp_X=YES
extra_netfs_types=NFS
dumpdev=NO
update_motd=NO
keyrate=fast
```

运行以下命令：

```
/usr/local/etc/rc.d/postgresql initdb
/usr/local/etc/rc.d/postgresql start
sockstat -l4
vi /var/db/postgres/data/pg_hba.conf
```

在 IPv4 部分添加以下配置：

```
# IPv4 local connections:
host    all             all             172.16.28.2/32          trust
```

运行以下命令：

```
/usr/local/etc/rc.d/postgresql restart
psql -h nextcloud.lab.bsd.pw -U postgres
CREATE USER nextcloud WITH PASSWORD 'something_random_from_bitwarden_or_other_password_manager';
CREATE DATABASE nextcloud TEMPLATE template0 ENCODING 'UNICODE';
ALTER DATABASE nextcloud OWNER TO nextcloud;
\q
vi /var/db/postgres/data/vacumm.sh
```

添加以下配置：

```
/usr/local/bin/reindexdb -a 1> /dev/null 2> /dev/null
/usr/local/bin/reindexdb -s 1> /dev/null 2> /dev/null
```

运行以下命令：

```
chmod +x /var/db/postgres/data/vacumm.sh
chown postgres:postgres /var/db/postgres/data/vacumm.sh
su - postgres -c 'crontab -e'
```

添加以下配置：

```
0 0 * * * /var/db/postgres/data/vacuum.sh
```

运行以下命令：

```
su - postgres -c 'crontab -l'
```

## Nginx

在 jail 中运行以下命令：

```
mkdir -p /usr/local/etc/nginx/ssl
cd /usr/local/etc/nginx/ssl
openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout nginx.key -out nginx.crt
```

输入 `server.key` 的密码短语：  

```
something_random_from_bitwarden_or_other_password_manager
```

使用你的最佳判断来填写密码提示后的提示信息。

运行以下命令：

```
chmod 400 nginx.key
ls -l
chown -R www:www /var/log/nginx
vi /usr/local/etc/nginx/nginx.conf
```

添加以下配置：

```
user                 www;
worker_processes     4;
worker_rlimit_nofile 51200;
error_log            /var/log/nginx/error.log;

events {
  worker_connections 1024;
}

http {
  include           mime.types;
  default_type      application/octet-stream;
  log_format        main  '$remote_addr - $remote_user [$time_local] "$request" ';
  access_log        /var/log/nginx/access.log main;
  sendfile          on;
  keepalive_timeout 65;

  upstream php-handler {
    server unix:/var/run/php-fpm.sock;
  }

  server {
    # ENFORCE HTTPS
    listen      80;
    server_name nextcloud.lab.bsd.pw;
    return      301 https://$server_name$request_uri;
  }

  server {
    listen              443 ssl http2;
    server_name         nextcloud.lab.bsd.pw;
    ssl_certificate     /usr/local/etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /usr/local/etc/nginx/ssl/nginx.key;

    # HEADERS SECURITY RELATED
    add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";

    # HEADERS
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;

    # PATH TO THE ROOT OF YOUR INSTALLATION
    root /usr/local/www/nextcloud/;

    location = /robots.txt {
      allow all;
      log_not_found off;
      access_log off;
    }

    location = /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location = /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }

    # BUFFERS TIMEOUTS UPLOAD SIZES
    client_max_body_size    16400M;
    client_body_buffer_size 1048576k;
    send_timeout            3000;

    # ENABLE GZIP BUT DO NOT REMOVE ETag HEADERS
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    location / {
      rewrite ^ /index.php$uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
      deny all;
    }

    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
      deny all;
    }

    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:$|/) {
      fastcgi_split_path_info ^(.+\.php)(/.*)$;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_param PATH_INFO $fastcgi_path_info;
      fastcgi_param HTTPS on;
      fastcgi_param modHeadersAvailable true;
      fastcgi_param front_controller_active true;
      fastcgi_pass php-handler;
      fastcgi_intercept_errors on;
      fastcgi_request_buffering off;
      fastcgi_keep_conn       off;
      fastcgi_buffers         16 256K;
      fastcgi_buffer_size     256k;
      fastcgi_busy_buffers_size 256k;
      fastcgi_temp_file_write_size 256k;
      fastcgi_send_timeout    3000s;
      fastcgi_read_timeout    3000s;
      fastcgi_connect_timeout 3000s;
    }

    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
      try_files $uri/ =404;
      index index.php;
    }

    # ADDING THE CACHE CONTROL HEADER FOR JS AND CSS FILES
    # MAKE SURE IT IS BELOW PHP BLOCK
    location ~ \.(?:css|js|woff|svg|gif)$ {
      try_files $uri /index.php$uri$is_args$args;
      add_header Cache-Control "public, max-age=15778463";
      # HEADERS SECURITY RELATED
      # IT IS INTENDED TO HAVE THOSE DUPLICATED TO ONES ABOVE
      add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
      # HEADERS
      add_header X-Content-Type-Options nosniff;
      add_header X-XSS-Protection "1; mode=block";
      add_header X-Robots-Tag none;
      add_header X-Download-Options noopen;
      add_header X-Permitted-Cross-Domain-Policies none;
      # OPTIONAL: DONT LOG ACCESS TO ASSETS
      access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg)$ {
      try_files $uri /index.php$uri$is_args$args;
      # OPTIONAL: DONT LOG ACCESS TO OTHER ASSETS
      access_log off;
    }
  }
}
```

运行以下命令：

```
vi /usr/local/etc/php/ext-20-pgsql.ini
```

添加以下配置：

```
[PostgresSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
```

运行以下命令：

```
vi /usr/local/etc/php/ext-30-pdo_pgsql.ini
```

添加以下配置：

```
[PostgresSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
```

运行以下命令：

```
touch /var/log/php-fpm.log
chown www:www /var/log/php-fpm.log
rm /usr/local/etc/php-fpm.d/www.conf
vi /usr/local/etc/php-fpm.d/www.conf
```

添加以下配置：

```
[www]
user = www
group = www
listen = /var/run/php-fpm.sock
listen.backlog = -1
listen.owner = www
listen.group = www
listen.mode=0660
pm = static
pm.max_children = 4
pm.start_servers = 2
pm.min_spare_servers = 2
pm.max_spare_servers = 4
pm.process_idle_timeout = 1000s;
pm.max_requests = 500
request_terminate_timeout = 0
rlimit_files = 51200
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

运行以下命令：

```
vi /usr/local/etc/php.ini
```

添加以下配置：

```
[PHP]
max_input_time=3600
engine = On
short_open_tag = On
precision = 14
output_buffering = OFF
zlib.output_compression = Off
implicit_flush = Off
unserialize_callback_func =
serialize_precision = 17
disable_functions =
disable_classes =
zend.enable_gc = On
expose_php = On
max_execution_time = 3600
max_input_time = 30000
memory_limit = 1024M
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
display_errors = Off
display_startup_errors = Off
log_errors = On
log_errors_max_len = 1024
ignore_repeated_errors = Off
ignore_repeated_source = Off
report_memleaks = On
track_errors = Off
html_errors = On
error_log = /var/log/php.log
variables_order = "GPCS"
request_order = "GP"
register_argc_argv = Off
auto_globals_jit = On
post_max_size = 16400M
auto_prepend_file =
auto_append_file =
default_mimetype = "text/html"
default_charset = "UTF-8"
doc_root =
user_dir =
enable_dl = Off
file_uploads = On
upload_max_filesize = 16400M
max_file_uploads = 64
allow_url_fopen = On
allow_url_include = Off
default_socket_timeout = 300
[CLI Server]
cli_server.color = On
[Date]
date.timezone = America/Denver
[filter]
[iconv]
[intl]
[sqlite3]
[Pcre]
[Pdo]
[Pdo_mysql]
pdo_mysql.cache_size = 2000
pdo_mysql.default_socket=
[Phar]
[mail function]
SMTP = localhost
smtp_port = 25
mail.add_x_header = On
[SQL]
sql.safe_mode = Off
[ODBC]
odbc.allow_persistent = On
odbc.check_persistent = On
odbc.max_persistent = -1
odbc.max_links = -1
odbc.defaultlrl = 4096
odbc.defaultbinmode = 1
[Interbase]
ibase.allow_persistent = 1
ibase.max_persistent = -1
ibase.max_links = -1
ibase.timestampformat = "%Y-%m-%d %H:%M:%S"
ibase.dateformat = "%Y-%m-%d"
ibase.timeformat = "%H:%M:%S"
[MySQLi]
mysqli.max_persistent = -1
mysqli.allow_persistent = On
mysqli.max_links = -1
mysqli.cache_size = 2000
mysqli.default_port = 3306
mysqli.default_socket =
mysqli.default_host =
mysqli.default_user =
mysqli.default_pw =
mysqli.reconnect = Off
[mysqlnd]
mysqlnd.collect_statistics = On
mysqlnd.collect_memory_statistics = Off
[OCI8]
[PostgreSQL]
pgsql.allow_persistent = On
pgsql.auto_reset_persistent = Off
pgsql.max_persistent = -1
pgsql.max_links = -1
pgsql.ignore_notice = 0
pgsql.log_notice = 0
[bcmath]
bcmath.scale = 0
[browscap]
[Session]
session.save_handler = files
session.save_path = "/tmp"
session.use_strict_mode = 0
session.use_cookies = 1
session.use_only_cookies = 1
session.name = PHPSESSID
session.auto_start = 0
session.cookie_lifetime = 0
session.cookie_path = /
session.cookie_domain =
session.cookie_httponly =
session.serialize_handler = php
session.gc_probability = 1
session.gc_divisor = 1000
session.gc_maxlifetime = 1440
session.referer_check =
session.cache_limiter = nocache
session.cache_expire = 180
session.use_trans_sid = 0
session.hash_function = 0
session.hash_bits_per_character = 5
url_rewriter.tags = "a=href,area=href,frame=src,input=src,form=fakeentry"
[Assertion]
zend.assertions = -1
[COM]
[mbstring]
[gd]
[exif]
[Tidy]
tidy.clean_output = Off
[soap]
soap.wsdl_cache_enabled=1
soap.wsdl_cache_dir="/tmp"
soap.wsdl_cache_ttl=86400
soap.wsdl_cache_limit = 5
[sysvshm]
[ldap]
ldap.max_links = -1
[mcrypt]
[dba]
[opcache]
opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
[curl]
[openssl]
```

再次升级 `rc.conf` 文件：

```
vi /etc/rc.conf
```

更新这些行，使其看起来像：

```
# DAEMONS | yes
syslogd_flags="-s -s"
sshd_enable=YES
postgresql_enable=YES
postgresql_class=postgres
postgresql_data=/var/db/postgres/data
php_fpm_enable=YES 
memcached_enable=YES
memcached_flags="-l 172.16.28.2"
nginx_enable=YES

# DAEMONS | no
sendmail_enable=NONE
sendmail_submit_enable=NO
sendmail_outbound_enable=NO
sendmail_msp_queue_enable=NO

# OTHER
clear_tmp_enable=YES
clear_tmp_X=YES
extra_netfs_types=NFS
dumpdev=NO
update_motd=NO
keyrate=fast
```

## Memcached

在 jail 中运行以下命令：

```
/usr/local/etc/rc.d/memcached start
/usr/local/etc/rc.d/php-fpm start
/usr/local/etc/rc.d/postgresql status
/usr/local/etc/rc.d/nginx start
sockstat -l4
ls -l /var/run/php-fpm.sock
mkdir -p /var/db/nextcloud/data
chown -R www:www /var/db/nextcloud
chown -R www:www /usr/local/www/nextcloud
```

在主机上用火狐浏览器打开网址  `https://172.16.28.2`

在 jail 上运行以下命令：

```
vi /etc/newsyslog.conf
```

将以下配置添加到文件的末尾：

```
/var/db/nextcloud/data/nextcloud.log    www:www 640     7       *       @T00    JC
/var/log/php-fpm.log                    www:www 640     7       *       @T00    JC
/var/log/nginx/error.log                www:www 640     7       *       @T00    JC
/var/log/nginx/access.log               www:www 640     7       *       @T00    JC
```
