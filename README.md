rtorrent-role
=========

This role installs rtorrent and ruTorrent on a system, configures and allows to use it as a service run by a dedicated user named rtorrent.

Requirements
------------
screen - in order to run rtorrent as detached program

Dependencies
------------
Roles chusiang.php7 and jdauphant.nginx are used to set up the webserver for rtorrent+rutorrent use.

Role Variables
--------------
Vars in use (vars/main.yml):
    
	libtorrent_pkg: libtorrent-0.13.6*, libtorrent-devel-0.13.6*
	rtorrent_pkg: rtorrent-0.9.6*
	rutorrent_repo: https://github.com/Novik/ruTorrent.git
	xmlrpc_pkg: xmlrpc-c
	curl_path: '/usr/bin/curl'
	stat_path: '/usr/bin/stat'

	rtorrent_user: rtorrent
	rtorrent_group: rtorrent
	scgi_port: 127.0.0.1:5000

	# php7 vars
	yum_php_version: '72u'
	centos_php_fastcgi_listen: 127.0.0.1:9000
	php_owner: nginx
	php_group: nginx 
	listen_port: 80
	var_www_directory: /var/www/rutorrent

	#nginx vars
	nginx_sites:
	  rutorrent:
	    - listen {{ listen_port }}
	    - server_name localhost
	    - root {{ var_www_directory }}
	    - location / {    try_files $uri $uri/ =404;  }
	    - |
	      location /RPC2 {
	        auth_basic "Restricted";
	        auth_basic_user_file /etc/nginx/auth_basic/users;
	        scgi_pass {{ centos_php_fastcgi_listen }};
	        include /etc/nginx/scgi_params;
	      }  
	    - |
	      location ~ .php$ {
	          fastcgi_split_path_info ^(.+\.php)(/.+)$;
	          include fastcgi_params;
	          fastcgi_pass {{ centos_php_fastcgi_listen }};
	          fastcgi_index index.php;
	          fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
	      }
	nginx_auth_basic_files:
	  users:
	    - rtorrent:$apr1$60sLjPnx$qB7Z7lYanaBgVHuBViBi8/

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: all
      connection: ssh
      become: yes
    
      roles:
        - rtorrent-role

License
-------

MIT

Author Information
------------------

slakaw9@github.com
