rails-deploy-procedure
======================

This is step by step procedure that I use to deploy a rails application on an Ubuntu VPS.  
Depending on the gems and versions, it may vary.

## Add user
### login with root

adduser rails
visudo

### Logout and login with user rails
mkdir apps; cd apps

## Install rvm and rails
\curl -sSL https://get.rvm.io | bash -s stable --rails
source /home/rails/.rvm/scripts/rvm
cd app/my_app
rvm install ruby-2.1.2
rvm use ruby-2.1.2

## Git
sudo apt-get install git
git clone https://username@bitbucket.org/username/my_app.git

## Postgres

http://www.cyberciti.biz/faq/howto-add-postgresql-user-account/
sudo apt-get install postgresql postgresql-contrib postgres-client
sudo su - postgres
psql template1
CREATE USER tom WITH PASSWORD 'myPassword';
CREATE DATABASE jerry;
GRANT ALL PRIVILEGES ON DATABASE jerry to tom;
\q

exit
sudo vim /etc/postgresql/9.3/main/pg_hba.conf

### Change peer to md5
local   all             postgres                                md5
local   all             all                                     md5

# Apache and passenger
sudo apt-get install apache libapache2-mod-passenger
sudo a2enmod passenger
sudo service apache2 restart
gem install passenger
sudo apt-get install libcurl4-openssl-dev apache2-threaded-dev libapr1-dev libaprutil1-dev
passenger-install-apache2-module

LoadModule passenger_module /home/rails/.rvm/gems/ruby-2.1.2/gems/passenger-4.0.53/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
  PassengerRoot /home/rails/.rvm/gems/ruby-2.1.2/gems/passenger-4.0.53
  PassengerDefaultRuby /home/rails/.rvm/gems/ruby-2.1.2/wrappers/ruby
</IfModule>

<VirtualHost *:80>
  ServerName yourhost.com
  ServerAlias www.yourhost.com
  # !!! Be sure to point DocumentRoot to 'public'!
  DocumentRoot /home/rails/apps/my_app/public
  <Directory /home/rails/apps/my_app/public>
     Require all granted
     AllowOverride None
     Allow from all
     Order allow,deny
     Options -MultiViews
  </Directory>
</VirtualHost>

sudo mv /var/www/html/index.html /var/www/html/index.html.old
sudo /etc/init.d/apache2 restart

# Node.js
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs

# Phantom.js
npm update
sudo npm install phantomjs -g

# RMagic
sudo apt-get install libmagickwand-dev
gem install rmagick -v '2.13.2'

# Testing from git and bundle
cd apps/my_app
bundle install
# Add gitignored files 
# Setup database
vim config/database.yml
rake db:migrate
raise s
# Move git version aside if everything works well
mv my_app my_app_git

# Deploy
# On server
vim .ssh/authorized_keys
# Add your public key from the local machine to authorized_keys on server
# Generate public key on server and add it to github/bitbucket deploy keys
ssh-keygen
cat .ssh/id_rsa.pub
# From local machine
cap deploy:setup
cap deploy:check
cap deploy

# Create shared files
cd apps/my_app/shared
mkdir config
vim database.yml
# Create production database in postgres
# rake db:migrate
