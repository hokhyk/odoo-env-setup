# source code installation:
## 1.  installing dependencies:
$sudo apt-get update
$sudo apt-get upgrade -y

$sudo apt-get install postgresql -y
$sudo su -c "createuser -s $(whoami)" postgres

$sudo apt-get install git python-pip python2.7-dev -y
$sudo apt-get install libxml2-dev libxslt1-dev libevent-dev libsasl2-dev libldap2-dev libpq-dev libpng12-dev libjpeg-dev poppler-utils node-less node-clean-css -y

$wget http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linuxjessie-amd64.deb
$sudo dpkg -i wkhtmltox-0.12.1.2_linux-jessie-amd64.deb
$sudo apt-get -fy install

$sudo -H pip install --upgrade pip
$wget https://raw.githubusercontent.com/odoo/odoo/10.0/requirements.txt
$sudo -H pip install -r requirements.txt

## 2. install Odoo
$ sudo adduser --disabled-password --gecos "Odoo" odoo
$ sudo su -c "createuser odoo" postgres
$ createdb --owner=odoo odoo-prod

### installing from Odoo source code
$ sudo su odoo
$ git clone https://github.com/odoo/odoo.git /home/odoo/odoo-10.0 -b 10.0 --depth=1

 # $ wget https://bootstrap.pypa.io/ez_setup.py -O - |sudo  python
$/home/odoo/odoo-10.0/odoo-bin --help
$exit

### setting up the configuration file
$sudo su -c "~/odoo-10.0/odoo-bin -d odoo-prod --save --stop-after-init" 
$sudo mkdir /etc/odoo
$sudo cp /home/odoo/.odoorc /etc/odoo/odoo.conf
$sudo chown -R odoo /etc/odoo

$sudo mkdir /var/log/odoo
$sudo chown odoo /var/log/odoo

#### configuring important parameters in odoo.conf
[options]
addons_path = /home/odoo/odoo-10.0/odoo/addons,/home/odoo/odoo-10.0/addons
admin_passwd = False
db_user = odoo-prod
dbfilter = ^odoo-prod$
logfile = /var/log/odoo/odoo-prod.log
proxy_mode = True
without_demo = True
workers = 3  #setting it to 1+2*P, where P is the number of processors. 
xmlrpc_port = 8069
#data_dir
#xmlrpc-interface

$sudo su -c "~/odoo-10.0/odoo-bin -c /etc/odoo/odoo.conf" 

## 3. setting up odoo server as a  system service
###  on ubuntu 16.04
$ sudo touch /lib/systemd/system/odoo.service
$sudo vi /lib/systemd/system/odoo.service
[Unit]
Description=Odoo
After=postgresql.service
[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/home/odoo/odoo-10.0/odoo-bin -c /etc/odoo/odoo.conf
[Install]
WantedBy=multi-user.target

$sudo systemctl enable odoo.service

$sudo systemctl odoo start|stop|status

### on Ubuntu 14.04 
$ sudo cp /home/odoo/odoo-10.0/debian/init /etc/init.d/odoo
$ sudo chmod +x /etc/init.d/odoo
$ cat /etc/init.d/odoo
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
DAEMON=/usr/bin/odoo
NAME=odoo
DESC=odoo
CONFIG=/etc/odoo/odoo.conf
LOGFILE=/var/log/odoo/odoo-server.log
PIDFILE=/var/run/${NAME}.pid
USER=odoo

$ sudo ln -s /home/odoo/odoo-10.0/odoo-bin /usr/bin/odoo
$ sudo chown -h odoo /usr/bin/odoo
$ sudo update-rc.d odoo defaults

$ sudo /etc/init.d/odoo start|stop|status or sudo service odoo start|stop|status|config

### checking odoo service from the command line
$curl http://localhost:8069
$curl -k https://localhost
<html><head><script>window.location = '/web' + location.hash;
</script></head></html>

$ sudo less /var/log/odoo/odoo-server.log
$ sudo tail -f /var/log/odoo/odoo-server.log

## 4. setting up a reverse proxy
$sudo apt-get install nginx
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo touch /etc/nginx/sites-available/odoo
$ sudo ln -s /etc/nginx/sites-available/odoo /etc/nginx/sites-enabled/odoo
$ sudo vi /etc/nginx/sites-available/odoo

upstream backend-odoo {
   server 127.0.0.1:8069;
}

server {
   location / {
      proxy_pass http://backend-odoo;
   }
}


$sudo nginx -t
$ sudo /etc/init.d/nginx reload

$curl http://localhost
<html><head><script>window.location = '/web' + location.hash;
</script></head></html>

### Enforcing HTTPS
$ sudo mkdir /etc/nginx/ssl && cd /etc/nginx/ssl
$ sudo openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
$ sudo chmod a-wx * # make files read only
$ sudo chown www-data:root * # access only to www-data group

$ vi /etc/nginx/sites-available/odoo
upstream backend-odoo {
   	server 127.0.0.1:8069;
}

upstream backend-odoo-im { server 127.0.0.1:8072; }

server {
	  listen 80;
	  server_name 118.190.201.241;
	  add_header Strict-Transport-Security max-age=2592000;
	#  rewrite ^/.*$ https://$host$request_uri? permanent;
	  return 301 https://$server_name$request_uri;
	#  rewrite ^/(.*)$ https://$server_name$request_uri? permanent;
}

server {
	  listen 443;
	  server_name 118.190.201.241;
	# ssl settings
	  ssl on;
	  ssl_certificate /etc/nginx/ssl/cert.pem;
	  ssl_certificate_key /etc/nginx/ssl/key.pem;
	  keepalive_timeout 60;
	# proxy header and settings
	  proxy_set_header Host $host;
	  proxy_set_header X-Real-IP $remote_addr;
	  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	  proxy_set_header X-Forwarded-Proto $scheme;
	  proxy_redirect off;

	# odoo log files
	access_log /var/log/nginx/odoo-access.log;
	error_log /var/log/nginx/odoo-error.log;
	# increase proxy buffer size
	proxy_buffers 16 64k;
	proxy_buffer_size 128k;
	# force timeouts if the backend dies
	proxy_next_upstream error timeout invalid_header http_500
	http_502 http_503;
	# enable data compression
	gzip on;
	gzip_min_length 1100;
	gzip_buffers 4 32k;
	gzip_types text/plain text/xml text/css text/less
	application/x-javascript application/xml application/json
	application/javascript;
	gzip_vary on;


	location / {
	   proxy_pass http://backend-odoo;
	}

	location ~* /web/static/ {
	# cache static data
	proxy_cache_valid 200 60m;
	proxy_buffering on;
	expires 864000;
	proxy_pass http://backend-odoo;
	}

	location /longpolling { proxy_pass http://backend-odoo-im;}
}


$sudo nginx -t
$ sudo /etc/init.d/nginx reload

## 5. server and module updates
### database backup, then swap odoo server to odoo-stage database and serving on 8080. 
#### way 1:
$ dropdb odoo-stage; createdb odoo-stage
$ pg_dump odoo-prod | psql -d odoo-stage
$ sudo su odoo
#### way 2:
$ createdb --template odoo-prod odoo-stage

#### finally:
$ cd ~/.local/share/Odoo/filestore/
$ cp -al odoo-prod odoo-stage
$ ~/odoo-10.0/odoo-bin -d odoo-stage --xmlrpc-port=8080 -c /etc/odoo/odoo.conf
$ exit

### then perform the updates on the original odoo server.
using git or just copying updated source files to the server directories.












