=============
How to use Putty and Filezilla to connect to an Openstack Virtual machine using putty and pageant
===================

Most users often want to use a command line prompt, and be able to transfer files to and from their desktop machine to the virtual Linux host they are using. Tools like Putty and Filezilla are commonly used for this purpose.

For Putty and for Filezilla to connect to an Openstack Virtual Machine, you need to generate a "key pair" - which you do with PuTTYgen. This will create a "public" and "Private" key.

The default settings should be fine (ssh-2 RSA) and 2048 length. Longer lengths are more secure.

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image1.png
    :align: center
    :alt:
Save the public key - you need to copy this key and paste it into the https://new-cloud.stfc.ac.uk . Login as usual to this Web interface, then click on your login id in the Top Right Hand Side corner:-

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image2.png
    :align: center
    :alt:


Select “SSH-key”. Paste your public key into the field displayed :-

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image3.png
    :align: center
    :alt:


Click on the Submit key.

In Putty Key Generator – Enter a Key passphrase – this is the password you need to unlock access to your private key. (Do not make this empty – it must have a password for security reasons!). Once filled in, click on the “save private key”. Save this file somewhere safe – and make a copy of it somewhere safe (like a USB stick). Take note of your password in whatever method you use to store passwords safely. (You now have an authentication system which comprises of “something you have” – your private key, and “something you know”, your password.

You can now use Putty to authenticate to any new VM’s that you create:-

1)	Download “pageant.exe” – it is a putty ssh agent and can be used to store your private key.
2)	Run pageant.exe – it will appear as an icon in your tool bar:-

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image4.png
    :align: center
    :alt:

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image5.png
    :align: center
    :alt:

(click on the arrow)
Right click on the icon in the toolbar (after clicking on the arrow), and select “add key”:-

Browse to where you private key is located – USB stick or local disk. You will be prompted for a password: This is the password to your private key.

Now start putty in the usual way:-

Enter the IP address of the virtual machine in the Hostname field:-

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image6.png
    :align: center
    :alt:


Click “Open” – you will be prompted for a username – (your username)  - the system will not ask for a password as it will use the ssh-agent we added your private key to earlier.
At this stage, you should be logged in!

Note that if you created a Virtual machine in Openstack before adding in your public key, you can add it afterwards from the console. This can be installed manually by pasting the key into the /home/<user name>/.ssh/authorized_keys file, where <user name> is your login name that you logged into the host with.

Filezilla (a tool for transferring files to and from a remote host) can be made to work in a similar way (i.e using your private key). Startup Filezilla, select File, then Site manager.

.. image:: /assets/howtos/PuttyAndFileZillaWithVMs/image7.png
    :align: center
    :alt:



Click on “new site” – fill in the details as per the example above, where Host= IP address of the virtual machine, Protocol = SFTP, Logon type = “Key file”, username –your username and the location of your private key file. Save the site details and then press “connect”: you should be prompted for a password of your private key. Once entered, you should be connected.

Note that in all of these examples, I have assumed that you can directly connect to the virtual machine. If you are using an “IPSEC based VPN” solution, then it should still all work.
