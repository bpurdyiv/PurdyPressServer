PurdyPressServer
================

Server Setup for PurdyPress

- nginx
- php5
- php5 zend Opcache (Alt to APC) Caches Php Code into memory
- memcache (Alt to Redis) Database and Object Caching Alleviate DB Load versions pecl/memcache pecl/memcached
- fastcgi_cache (Alt to Varnish)
- W3 Total Cache (Wordpress Plugin)
- Nginx (Wordpress Plugin)

Links
- [php, mysql, postfix, nginx](https://rtcamp.com/tutorials/linux/ubuntu-php-mysql-nginx-postfix/)
- [wordpress, nginx server minimal](https://rtcamp.com/wordpress-nginx/tutorials/single-site/minimal/)
- [php zend opcache & webview](https://rtcamp.com/tutorials/php/zend-opcache/)
- [memcache](https://rtcamp.com/tutorials/php/memcache/)
- [fastcgi](https://rtcamp.com/wordpress-nginx/tutorials/single-site/fastcgi-cache-with-purging/)
- [W3 Total Cache](https://rtcamp.com/wordpress-nginx/tutorials/single-site/w3-total-cache/)


## Initial Configs

1. Create New User
``` sh
$ adduser newUserName
```
2. Add new User to Sudoers
``` sh
$ visudo
```
  * Add following line below ``` root ALL+(ALL:ALL) ALL ```

  ```
  newUserName ALL=(ALL:ALL) ALL
  ```

3. Reconfig SSH - Change respective setting to those below...
```sh
$ vi /etc/ssh/sshd_config
```
  * Change Port Number
    ```
    Port NewPortNumber
    ```

  * Disable Root Login
    ```
    PermitRootLogin no
    ```

  * Only allow specific users
    ```
    UseDNS no
    AllowUsers newUserNmae
    ```

  * Save and exit, then reload ssh
    ``` sh
    $ reload ssh
    ```

4. Reset Timezone
  ``` sh
  $ dpkg-reconfigure tzdata
  ```

5. Update System
  ``` sh
  $ apt-get update
  ```

  ``` sh
  $ apt-get upgrade
  ```

  ``` sh
  $ apt-get dist-upgrade
  ```

## System Installs & Configs

1. Install and Config Firewall
``` sh
$ apt-get install ufw
```

``` sh
$ ufw default deny incoming
```

``` sh
$ ufw default allow outgoing
```

``` sh
$ ufw allow 80/tcp
```

``` sh
$ ufw allow 443/tcp
```

``` sh
$ ufw allow youSSHPortNumber/tcp
```

``` sh
$ ufw enable
```

2. Install and Config Postfix
``` sh
$ apt-get install postfix mailutils
```
  * Edit Postfix config file
    ``` sh
    $ vi /etc/postfix/main.cf
    ```

    * Change ``` myhostname = localhost ``` to
    ```
    myhostname = YourMailServerHostName #ie. mail.exampleserver.com
    ```

    * Restart Postfix
    ``` sh
    $ service postfix restart
    ```

3. Install and Config Fail2Ban
  * Install Fail2Ban
    ``` sh
    $ apt-get install fail2ban
    ```

  * Config Fail2Ban
    * Copy jail.conf to jail.local
      ``` sh
      $ cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
      ```

    * Edit jail.local
      ``` sh
      $ vi /etc/fail2ban/jail.local
      ```

      * Change Destemail
        ```
        destemail = your@email.com
        ```

      * Change Action replace ``` action = %(action_)s ``` to
        ```
        action = %(action_mwl)s
        ```

    * Restart Service
      ``` sh
      $ service fail2ban restart
      ```

## Install Main Stack
1. Install misc
``` sh
$ apt-get install curl git-core python-software-properties
```

2. Install NginX
  * Add NginX Repo
    ``` sh
    $ add-apt-repository ppa:nginx/stable
    ```

  * Update Apt
    ``` sh
    $ apt-get update
    ```

  * Install NginX
  ```
  $ apt-get install nginx
  ```

3. Install PHP 5.5
  * Add PHP Repo
    ``` sh
    $ add-apt-repository ppa:ondrej/php5
    ```

  * Update Apt
    ``` sh
    $ apt-get update
    ```

  * Install PHP and modules
    ``` sh
    $ apt-get install php5-common php5-fpm php5-mysqlnd php5-gd php5-xmlrpc php5-curl  php5-cli php-pear php5-imap php5-mcrypt php5-tidy
    ```

4. Install MySQL
  * Using MariaDB instead of MySQL... To add current repo follow directions on [their site](https://downloads.mariadb.org/mariadb/repositories/). Current instructions as of 4/1/14 for version 10 below
    * Add Apt Key
      ``` sh
      $ apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
      ```

    * Add Repo
      ``` sh
      $ add-apt-repository 'deb http://mirrors.syringanetworks.net/mariadb/repo/10.0/ubuntu precise main'
      ```

    * Update Apt
      ``` sh
      $ apt-get update
      ```

    * Install
      ``` sh
      $ apt-get install mariadb-server
      ```

    * Run MySQL Secure Installation
      ``` sh
      $ mysql_secure_installation
      ```

5. Install Rmate for editing with Sublime Text 3 (as of 4/1/14) https://github.com/henrikpersson/rsub https://github.com/aurora/rmate

  * Paste the following in the ssh config file on local machine
    ```
    Host your_remote_server.com
        RemoteForward 52698 127.0.0.1:52698
    ```

  * ‘Install’ the rsub remote script
    ``` sh
    $ sudo wget -O /usr/local/bin/rsub https://raw.github.com/aurora/rmate/master/rmate
    ```

  * Make that script executable
    ``` sh
    $ sudo chmod +x /usr/local/bin/rsub
    ```

  * Lastly, run rsub on the remote file you want to edit locally
    ``` sh
    $ rsub ~/my_project/my_file.html
    ```


## Configure Shit
1. Configure Zend Opcache https://rtcamp.com/tutorials/php/zend-opcache/
  * In your php.ini or /etc/php5/fpm/conf.d/05-opcache.ini
    ```
    zend_extension=opcache.so
    opcache.memory_consumption=512
    opcache.max_accelerated_files=50000

    ;following can be commented for production server
    opcache.revalidate_freq=0
    opcache.consistency_checks=1
    ```

  * Install a Webviewer... or maybe New Relic

2. Install and config Memcache https://rtcamp.com/tutorials/php/memcache/
  * Install memcached and php5-memcache
    ``` sh
    $ apt-get install memcached php5-memcache
    ```

  * Restart php5-fpm
    ``` sh
    $ service php5-fpm restart
    ```

  * Config  Max Memory
    * Open config File
      ```
      $ vi /etc/memcached.conf
      ```

    * Look for ``` -m 64 ``` Change to desired max memory ie. ``` -m 512 ```
    * Restart memcached
      ``` sh
      $ service memcached restart
      ```

  * Move PHP’s session storage to memcache
    * Open php-memcache config
      ``` sh
      $ vi /etc/php5/mods-available/memcache.ini
      ```

    * Add following lines
      ```
      session.save_handler = memcache
      session.save_path = "tcp://localhost:11211"
      ```

  * In w3-total-cache plugin you need to choose memcache as backend for object-cache and database-cache.
  * Install web viewer or maybe New Relic

3. Configure fastcgi_cache with conditional purging
  * No install... build into Nginx
  * Need to install wordpress plugin to the nginx to purge cache when needed...
    * http://wordpress.org/plugins/nginx-helper/
    * Check "Enable Cache Purge"
  * NginX needs a folder to store cache. Below we will use ``` /var/run ``` it's a Ram Disk (tmpfs) folder. To check available storage use following command
    ``` sh
    $ df -h /var/run
    ```

  * Will need to play with amount of system memory needed for this to work. If You do not have enough memory, then use a regular disk directory for cache storage....
  * Below is a NginX server config using the fastcgi_cache.
  ```
  #move next 3 lines to /etc/nginx/nginx.conf if you want to use fastcgi_cache across many sites
  fastcgi_cache_path /var/run/nginx-cache levels=1:2 keys_zone=WORDPRESS:100m inactive=60m;
  fastcgi_cache_key "$scheme$request_method$host$request_uri";
  fastcgi_cache_use_stale error timeout invalid_header http_500;
  fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
  server {
  	server_name example.com www.example.com;

  	access_log   /var/log/nginx/example.com.access.log;
  	error_log    /var/log/nginx/example.com.error.log;

  	root /var/www/example.com/htdocs;
  	index index.php;

  	set $skip_cache 0;

  	# POST requests and urls with a query string should always go to PHP
  	if ($request_method = POST) {
  		set $skip_cache 1;
  	}
  	if ($query_string != "") {
  		set $skip_cache 1;
  	}

  	# Don't cache uris containing the following segments
  	if ($request_uri ~* "/wp-admin/|/xmlrpc.php|wp-.*.php|/feed/|index.php|sitemap(_index)?.xml") {
  		set $skip_cache 1;
  	}

  	# Don't use the cache for logged in users or recent commenters
  	if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in") {
  		set $skip_cache 1;
  	}

  	location / {
  		try_files $uri $uri/ /index.php?$args;
  	}

  	location ~ .php$ {
  		try_files $uri /index.php;
  		include fastcgi_params;
  		fastcgi_pass unix:/var/run/php5-fpm.sock;

  		fastcgi_cache_bypass $skip_cache;
  	        fastcgi_no_cache $skip_cache;

  		fastcgi_cache WORDPRESS;
  		fastcgi_cache_valid  60m;
  	}

  	location ~ /purge(/.*) {
  	    fastcgi_cache_purge WORDPRESS "$scheme$request_method$host$1";
  	}

  	location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
  		access_log off;	log_not_found off; expires max;
  	}

  	location = /robots.txt { access_log off; log_not_found off; }
  	location ~ /\. { deny  all; access_log off; log_not_found off; }
  }
  ```
  
  * The line fastcgi_cache_path is where cache is stored. If limited memory use a disk directory.
  * The line fastcgi_cache_use_stale is what makes caching on Nginx-side unique. This line tells Nginx to use old (stale) cached version of page if PHP crashes. This is something not possible with WordPress caching plugins.
  * Test and reload nginx
    ```
    $ nginx -t && service nginx reload
    ```

4. Install and Config W3 Total Cache.
  * https://rtcamp.com/wordpress-nginx/tutorials/single-site/w3-total-cache/
