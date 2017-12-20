ubuntu16.04
vagrant vagrant

安装python2.7.9环境：
$sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm

$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash

pyenv install --list
pyenv versions
which python
python -V
$ pyenv install -v 2.7.9
pyenv rehash

# 创建新的环境,位于 ~/.pyenv/versions/
$ pyenv virtualenv 2.7.9 2.7.9env

# 切换到新的环境
$ pyenv activate 2.7.9env

# 退回到系统环境
$ pyenv deactivate

# 删除新创建的环境
$ rm -rf ~/.pyenv/versions/2.7.9env/

安装postgresql9.4:
安装前的检查,首先查看是否已经安装了旧版本：
dpkg -l |grep postgresql
如果已经安装了某个版本的postgresql，请先卸载。
添加postgresql源：
sudo touch /etc/apt/sources.list.d/pgdb.list
sudo vim /etc/apt/sources.list.d/pgdb.list
把下面这行数据添加到pgdb.list文件中：
deb https://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main
执行下面的命令添加postgresql安装包的秘钥：
sudo wget --quiet -O - https://postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - 
接下来就可以安装了：
sudo apt-get update
sudo apt-get install postgresql-9.4
配置文件目录：/etc/postgresql/9.4/main/
配置远程链接：
sudo vi /etc/postgresql/9.4/main/pg_hba.conf
增加host  all    all    192.168.0.0/24    md5
sudo vi /etc/postgresql/9.4/main/postgresql.conf
增加：listen_addresses = '*'
重启postgresql服务：
sudo /etc/init.d/postgresql restart
修改postgres密码：
$sudo su postgres
$psql
postgres=# \password postgres
Enter new password:  postgres
Enter it again: postgres
postgres=# 
创建odoo数据库用户：
sudo su - postgres -c "createuser -s odoo"
sudo su - postgres
$psql
postgres=# \password odoo
Enter new password:  odoo
Enter it again: odoo
postgres=# \q


安装Odoo源码：
# apt-get update && apt-get upgrade # Install system updates
# apt-get install sudo # Make sure 'sudo' is installed

# useradd -m -g sudo -s /bin/bash odoo # Create an 'odoo' user
with sudo powers
# passwd odoo # Ask and set a password for the new user

$ sudo apt-get update && sudo apt-get upgrade #Install system
updates
$ sudo apt-get install git # Install Git

mkdir ~/odoo-dev # Create a directory to work in
cd ~/odoo-dev # Go into our work directory
git clone https://github.com/odoo/odoo.git -b 10.0 --depth=1       # Get Odoo source code

本环境实际地址：/home/vagrant/share/odoo-dev
安装python依赖：
cd ~/share/odoo-dev
$ pyenv activate 2.7.9env

echo -e "\n---- Install tool packages ----"
sudo apt-get install wget git python-pip python-imaging python-setuptools python-dev libxslt-dev libxml2-dev libldap2-dev libsasl2-dev node-less postgresql-server-dev-all -y

pip install watchdog   #for development and debug perpose.
pip install -r requirements.txt

$ sudo apt-get install npm                     # Install NodeJs and its package manager
$ sudo ln -s /usr/bin/nodejs /usr/bin/node      # call node runs nodejs
$ sudo npm install -g less less-plugin-clean-css    #Install less compiler

echo -e "\n---- Install wkhtml and place on correct place for ODOO 8 ----"
sudo wget http://download.gna.org/wkhtmltopdf/0.12/0.12.2.1/wkhtmltox-0.12.2.1_linux-trusty-amd64.deb
sudo dpkg -i wkhtmltox-0.12.2.1_linux-trusty-amd64.deb
sudo apt-get install -f -y
sudo dpkg -i wkhtmltox-0.12.2.1_linux-trusty-amd64.deb
sudo cp /usr/local/bin/wkhtmltopdf /usr/bin
sudo cp /usr/local/bin/wkhtmltoimage /usr/bin


最后配置odoo的启动配置文件：
cd ~/share/odoo-dev/
mkdir customedaddons
chmod 755 customedaddons
touch ~/share/odoo-dev/odoo-server.conf
vi odoo-server.conf
录入以下配置：
[options]
admin_passwd = qazwsx!@#$
db_host = 127.0.0.1
db_port = 5432
db_user = odoo
db_password = odoo
#dbfilter = ^%h$
addons_path = ./addons/,./customedaddons
log-level = debug
--dev=all
--save

To start an Odoo server instance, just run:
$ python odoo-bin -c odoo-server.conf

$ sudo service odoo restart






