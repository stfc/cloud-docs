##################################################
Create additional user(s) on OpenStack VM/Instance
##################################################

Introduction
############

This guide explains how to create users in Linux using the command line and the _useradd_ command and  grant administrator privileges using _sudo_.

The `sudo` command provides a mechanism for granting administrator privileges — ordinarily only available to the root user — to normal users. 

This guide will show you how to create a new user with `sudo` access on Linux, without having to modify your server’s `/etc/sudoers` file.

Prerequisites
#############

* Valid Login and SSH or Consiole Access to VM
* Have sudo previllages
* VPN (if outside the network)

Adding Users
############

#. Logging into you server

    SSH in to your server as using your FEDADMIN user:


    .. code-block:: shell

      %ssh FEDID@172.16.113.47
        ____ _____ _____ ____    ____ _                 _
      / ___|_   _|  ___/ ___|  / ___| | ___  _   _  __| |
      \___ \ | | | |_ | |     | |   | |/ _ \| | | |/ _` |
        ___) || | |  _|| |___  | |___| | (_) | |_| | (_| |
      |____/ |_| |_|   \____|  \____|_|\___/ \__,_|\__,_|

                Location: r89.harwell.europe hpd r89rack241
                  Branch: xfu59478/mr_build_centos_aq_image (sandbox)
              Archetype: cloud
            Personality: nubesvms
        Operating System: centos7x-x86_64
          Snapshot Date: 2019-05-20

#. Login as Root

    Use the `sudo` command to login in as root:

    .. code-block:: shell

      [FEDID@preprod-horizon1 ~]$ sudo su -

      [root@preprod-horizon1 ~]#

#. Adding a New User to the System using the script (addusers.sh) below. Edit teh scipt and save it.

    .. code-block:: shell

      declare -A KEYS

      KEYS[ntint]="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOXk3OukWtXQHbyhxuG2ST/zhGfoPHQi/5Zvt5PELskztdPMZRKn3bCqSk0nZQppoFgGsze93CWcfiY2ppUTGoKbZMnmDSjfO0DUFXB/ADNJapo53Yd0lRnxM45JuFT+AS8cg/TalzbBeXcO2Tyr/hFTm2p7b5PQZgny9RwdXI6W0rTyxnktBeMoy6bMepPueZgUhJVJIKFaAymvVlmeZHhlRX2MB/Y7OTpETNiuTbVaSw3kI57rZ16pgvCWwl0RyQon9V6yBsduaF9MAHTVlsuKBd7S7Q1V/19CpaAKdowq+HrZ5lbu3H7+GvX8LSpNAGzOtuNOxm50U/jcOWaUHxMtNCTMm1oHtdN13vADtTpU5MbtzJ89q/pSTfJVbX0oj8AFN+gtb4hlvKWaWwEUwOe5P4JlVIX4PjHZKzAdf+s1cumJ9sbjgvFIKzrGes6lpUBdjw0WGQnB0GLSkml7Hu6x8jecd4hC04tSqgJLw+KXZysVYCUkIuDEp/GKGv8aM= ntint@byod-vpn-168-0067.rl.ac.uk"

      KEYS[mtint]="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOXk3OukWtXQHbyhxuG2ST/zhGfoPHQi/5Zvt5PELskztdPMZRKn3bCqSk0nZQppoFgGsze93CWcfiY2ppUTGoKbZMnmDSjfO0DUFXB/ADNJapo53Yd0lRnxM45JuFT+AS8cg/TalzbBeXcO2Tyr/hFTm2p7b5PQZgny9RwdXI6W0rTyxnktBeMoy6bMepPueZgUhJVJIKFaAymvVlmeZHhlRX2MB/Y7OTpETNiuTbVaSw3kI57rZ16pgvCWwl0RyQon9V6yBsduaF9MAHTVlsuKBd7S7Q1V/19CpaAKdowq+HrZ5lbu3H7+GvX8LSpNAGzOtuNOxm50U/jcOWaUHxMtNCTMm1oHtdN13vADtTpU5MbtzJ89q/pSTfJVbX0oj8AFN+gtb4hlvKWaWwEUwOe5P4JlVIX4PjHZKzAdf+s1cumJ9sbjgvFIKzrGes6lpUBdjw0WGQnB0GLSkml7Hu6x8jecd4hC04tSqgJLw+KXZysVYCUkIuDEp/GKGv8aM= mtint@byod-vpn-168-0067.rl.ac.uk"

      groupadd sshusers

      for FEDID in "${!KEYS[@]}"
        do
          SSH_PUBLIC_KEY=${KEYS[$FEDID]}
          echo "Creating USER $FEDID"
          echo "Adding KEY $SSH_PUBLIC_KEY"
          echo " "
          echo -n "Add $FEDID to sudo group (y/n)? "
          read answer
          if [ "$answer" != "${answer#[Yy]}" ] ;then
            useradd $FEDID -g wheel;
            echo "$FEDID ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers;
          else
          useradd $FEDID;
          fi
          usermod -a -G sshusers $FEDID
          mkdir /home/$FEDID/.ssh
          echo "$SSH_PUBLIC_KEY" >> /home/$FEDID/.ssh/authorized_keys
          chown -R $FEDID /home/$FEDID/.ssh
          echo "User USER $FEDID Created"
          echo " "
      done
          chmod 0440 /etc/sudoers

This script should only be run once. 

User (FEDID) and SSH KEYS are stored in an Array with the format KEY[FEDID]="SSH KEY"

.. code-block:: shell

  KEYS[cog12345]="ssh-rsa xxxxx"

After editing the script with users and keys run by typing

.. code-block:: shell
  
  source addusers.sh

Testing
=======

Test by ssh to the server 

.. code-block:: shell

    ssh mtint@172.16.113.47

    Last login: Thu Feb 25 23:10:27 2021 from vpn-3-065.rl.ac.uk
      ____ _____ _____ ____    ____ _                 _
    / ___|_   _|  ___/ ___|  / ___| | ___  _   _  __| |
    \___ \ | | | |_ | |     | |   | |/ _ \| | | |/ _` |
      ___) || | |  _|| |___  | |___| | (_) | |_| | (_| |
    |____/ |_| |_|   \____|  \____|_|\___/ \__,_|\__,_|


              Location: r89.harwell.europe hpd r89rack241
                Branch: xfu59478/mr_build_centos_aq_image (sandbox)
            Archetype: cloud
          Personality: nubesvms
      Operating System: centos7x-x86_64
        Snapshot Date: 2019-05-20

      Notice: Core software packages have been updated, a system reboot is required for them to take effect.

    %id mtint
    uid=1002(mtint) gid=10(wheel) groups=10(wheel),1001(sshusers)

Check sudo privilege

.. code-block:: shell
  
  sudo su -
  Last login: Thu Feb 25 22:09:53 UTC 2021 on pts/1
    Notice: Core software packages have been updated, a system reboot is required for them to take effect.

  [root@preprod-horizon1 ~]#

Conclusion
==========

You have now sucessfully created user(s) on COG OpenStack VM/Instance.

If your having problems :ref:`contact_email`