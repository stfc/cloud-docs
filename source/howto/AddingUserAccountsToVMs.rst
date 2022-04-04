==============================================================
Adding multiple user accounts to a VM
==============================================================

#########
Outline
#########
This script will add a list of fed IDs or openstack account names to the VM it is run on. 
Make sure that all the accounts you are attempting to add exist within the Openstack project the VM is in.

#########
Method
#########
Here is the script. Save this as `addusers-ubuntu.sh` to the VM.

.. code-block:: bash
        
        #!/bin/bash

        FEDIDS=$(cat $1)
        for fedID in $FEDIDS
        do
                $(echo -e "\n\nn" | adduser $fedID -q --gecos ",,,,") &>/dev/null
        done

To run this script, first create a file called `fedIDs.txt` containing the fed IDs you want to add as user accounts. Seperate each ID with a new line.
Then, run the script with sudo (you will need sudo privileges to add users) as follows, passing the file containing the fedIDs as an argument:

.. code-block:: bash

        sudo bash addusers-ubuntu.sh fedIDs.txt

To check that the users have been added correctly, you can check the contents of the `/etc/passwd` file. You should see the new accounts at the bottom of this file, e.g.

.. code-block:: bash

        $ tail -n 1 /etc/passwd
        <fedID>:x:1008:1006:,,,:/home/<fedID>:/bin/bash

==============================================================
Adding a user account to multiple cloud hosts within a project
==============================================================

#########
Outline
#########
For some types of Openstack projects, it’s common to have a list of user accounts, that may or may not be external users, that need ssh access to a (large) number of hosts within that project.
This document provides an outline approach as to how to approach this task.

#########
Method
#########
Make sure you have root access to the systems, or at the very least, have a login to each system with sudo access. If you are using a ssh key pair to login (the preferred method within STFC), make sure you have your private key loaded on a ssh-agent and ensure that “Agent forwarding” feature is enabled.
If this is a common task that you have to do, it is worth using the same “command and control” hosts in all of your operations for running scripts on the hosts within projects.
Step 1 – Find the list of hosts within a project
Find the project from:
Openstack project list | grep –v 844e | grep –v rally
…from there, you can list all of the virtual machines that are in a particular project:-
Openstack server list –project “Rucio at RAL”
This will return a list of hostnames and their IP addresses (and float IP addresses). You can make this just a list of IP addresses using something like:-
openstack server list --project "Rucio at RAL" | awk -F\| '{ print $5 }' | grep 'Internal' | sed 's/^ Internal=//'
….which just returns a list of the 172.16.x.y network IP addresses – there is no point in returning the 192.168.x.y IP addresses as you cannot directly ssh to these anyway, but any float IP addresses, you could append as needed.
Output this to a file “all_IP_addresses.text”.
Check login to each host, identify host and check which hosts have users enabled on them
