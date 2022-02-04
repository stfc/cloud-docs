======================================================================================================
Grow the disk of an Openstack VM by cloning the machine
======================================================================================================

##########
Scenario
##########
In the STFC cloud, if you created a new VM in instances, the default setting in the :guilabel:`Sources` was to create the VM with a image. If you run out of disk space on the VM, you can usually :guilabel:`resize` it to a new Flavor, and you will have the same VM, but with more space.

.. note::

    See :ref:`Resize_VM` on how to resize a VM.

You can also create a new instance with :guilabel:`Volume` as :guilabel:`Sources`. However, with :guilabel:`Volumes`, the system ignores the disk sizing aspect of the :guilabel:`flavor`. This document describes how to create a clone of the original VM and increase the disk size, so that space is available on the new VM.

###########
Method
###########

WebUI
********************
.. note::

    Check your Quotas for our project: Volume sizes, number of volumes, and number of volume snapshots are all parameters for which your project has a quota for, and there is a fair chance that what you have allocated may not be enough. Check with a cloud admin first before proceeding if you are unsure.

1. Go to :guilabel:`Compute` → :guilabel:`Instances` in the LHS menu
2. Click the drop-down menu on the right-hand side (in :guilabel:`Actions` column) and select :guilabel:`SHUT OFF INSTANCE`

.. image:: /assets/howtos/GrowDisk/GrowDisk-1.png
    :align: center
    :alt:

3. Click on the instance name

.. image:: /assets/howtos/GrowDisk/GrowDisk-2.png
    :align: center
    :alt:

4. Check the :guilabel:`Volumes Attached` for the volume attached to `/dev/vda` and click on the volume name

.. image:: /assets/howtos/GrowDisk/GrowDisk-3.png
    :align: center
    :alt:

5. Open the drop-down menu on the top left corner and click :guilabel:`CREATE SNAPSHOT`

.. image:: /assets/howtos/GrowDisk/GrowDisk-4.png
    :align: center
    :alt:

6. Give it a name and click :guilabel:`CREATE VOLUME SNAPSHOT (FORCE)`

.. image:: /assets/howtos/GrowDisk/GrowDisk-5.png
    :align: center
    :alt:

.. note::

    Once the volume is attached to a VM, you cannot remove it: the only way to remove the VM host from the volume is if the instance is created, and when the VM was created, the setting for the volume to persist still exists.

7. Go to :guilabel:`Volume` → :guilabel:`Volumes` in the LHS menu, then click on :guilabel:`+ CREATE VOLUME`

.. image:: /assets/howtos/GrowDisk/GrowDisk-6.png
    :align: center
    :alt:

8. Give it a name an in :guilabel:`Volume Source` Select :guilabel:`SNAPSHOT`; In :guilabel:`Use snapshot as a source` select the snapshot you created and input the :guilabel:`Size (GiB)` you need. After that, Click :guilabel:`CREATE VOLUME`

.. image:: /assets/howtos/GrowDisk/GrowDisk-7.png
    :align: center
    :alt:

9. Refer to :ref:`VM_Create_WebUI` except in :guilabel:`Source` you should select :guilabel:`Select Boot Source` → :guilabel:`Volume` and select the volume you created from the snapshot

.. note::

    The flavor of the new VM does not affect the disk size – only the volume parameters does this !

.. image:: /assets/howtos/GrowDisk/GrowDisk-8.png
    :align: center
    :alt:

.. note::

    When creating a new instance from a volume, the volume may still have previous users created in the system and their home directories and they may have their details still in the ``/etc/sudoers.d/cloud`` file. However, you will not find that your :guilabel:`FEDID` has been added to the ``/etc/sudoers.d/cloud`` file.
