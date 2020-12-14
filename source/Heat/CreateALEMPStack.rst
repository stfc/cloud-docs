#################
Create LEMP Stack
#################

LEMP Stack consists of:

  - Linux
  - Nginx
  - MySQL
  - PHP


Bash Script
###########

A bash script can be added to a stack template so that when a virtual machine is
built, it will automatically set up the VM with the components for a LEMP stack.
This script also shows how to automate mysql_secure_installation as well.

.. code-block:: bash

  #!/bin/bash -v
  apt-get update
  # packages for PAM should not be updated by using a bash script
  # these return prompts to ask if the configuration on the VM can be changed
  # and the script becomes 'stuck'
  #To overcome this, use apt-mark hold <packages>
  apt-mark hold libpam-krb5 libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpam0g
  apt-get -y upgrade
  #Then unhold the packages
  apt-mark unhold libpam-krb5 libpam-modules libpam-modules-bin libpam-runtime libpam-systemd libpam0g

  #Install nginx
  apt-get install -y nginx

  # In this example, we will install and set up mysql. Alternatively this step can be done after VM creation
  apt-get install -y expect #to run intereactive script inside bash shell

  #Install mySQL and set up root access
  apt-get install -y mysql-server
  SECURE_MYSQL=$(expect -c "
  set timeout 5
  spawn mysql_secure_installation

  expect \"Press y|Y for Yes, any other key for No:\"
  send \"n\r\"

  expect \"Please set the password for root here.\"
  send \"temporarypw\r\"

  expect \"Re-enter password:\"
  send \"temporarypw\r\"

  expect \"Remove anonymous users? (Press y|Y for Yes, any other key for No):\"
  send \"y\r\"

  expect \"Disallow root login remotely? (Press y|Y for Yes, any other key for No):\"
  send \"y\r\"

  expect \"Remove test database and access to it? (Press y|Y for Yes, any other key for No):\"
  send \"y\r\"

  expect \"Reload privilege tables now? (Press y|Y for Yes, any other key for No)\"
  send \"y\r\"
  expect eof
  ")
  echo "$SECURE_MYSQL"

  mysql <<EOF

  SELECT user,authentication_string,plugin,host FROM mysql.user;
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'secret';
  FLUSH PRIVILEGES;
  SELECT user,authentication_string,plugin,host FROM mysql.user;

EOF

  #add-apt-repository universe #already available for ubuntu bionic
  #install PHP and PHP packages
  apt-get install -y php-fpm php-mysql

  echo "All components for LEMP stack have been installed!"


> This script sets up the root password as a temporary password. If you set up a temporary password, this **must** be changed once the root user has signed into mySQL.
 Please see the documentation **Installing and setting up MySQL database in a Stack** for information about how to change the root password.



References
##########

https://gist.github.com/Mins/4602864

https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-ubuntu-18-04
