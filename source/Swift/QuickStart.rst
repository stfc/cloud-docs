.. _swift_quick_start:

==========================
Object Storage Quick Start
==========================


.. _swift_create_container:

Create a Container
------------------

In the Openstack web interface, expand `Object Store` and then click `Containers` on the left hand side of the screen.

.. image:: /assets/howtos/CreateAContainer/1-menu.png

Click `+ Container`.

.. image:: /assets/howtos/CreateAContainer/2-create-container.png

On the `Create Container` screen, enter a name for the container and select whether the container should be publicly accessible, then click `Submit`.

.. image:: /assets/howtos/CreateAContainer/3-create-container-form.png

.. warning::

  Selecting "public" will allow the contents of a container to be viewed by anyone, with no authentication required.
  See :ref:`swift_cli_update_metadata` or :ref:`swift_sdk_containers` for greater control over read and write access.

If successful, the container will be listed below the `+ Container` button.

.. image:: /assets/howtos/CreateAContainer/5-container-list.png

Details of the container may be viewed by clicking on the listing. This will also display a checkbox that may be clicked to enable or disable public access.

.. image:: /assets/howtos/CreateAContainer/6-container-details.png


.. _swift_create_object:

Upload a File
-------------

Having selected a listed container, click the `Upload` icon next to `+ Folder`.

.. image:: /assets/howtos/CreateAContainer/7-object-upload.png

Select a file to upload, and a unique name for it in the container. Then click `Upload File`.

.. image:: /assets/howtos/CreateAContainer/8-object-form.png

.. warning::

  Files uploaded in this way must be less than 5GiB. See :ref:`swift_cli_large_objects` for instructions on uploading larger files.
  If `Error: Unable to upload the object` is displayed for a file smaller than this, you may need to contact the Cloud team to request a quota increase.


If successful, the file will be listed below the `Upload` icon.

.. image:: /assets/howtos/CreateAContainer/10-object-list.png

To view file details, select the arrow next to `Download` to view the options, then click `View Details`.

.. image:: /assets/howtos/CreateAContainer/11-object-options.png

The file details will then be shown.

.. image:: /assets/howtos/CreateAContainer/12-object-details.png


.. _swift_create_folder:

Create a Folder
---------------

To create an empty folder, click `+ Folder`.

.. image:: /assets/howtos/CreateAContainer/13-folder-create.png

Enter a unique folder name, then click `Create Folder`.

.. image:: /assets/howtos/CreateAContainer/14-folder-form.png

Alternatively, folders may be created by including a new folder name, followed by a delimiter ('/'), before the name of an uploaded file.

.. image:: /assets/howtos/CreateAContainer/16-folder-form-alt.png

If successful, the folder will be listed with other files in the container.

.. image:: /assets/howtos/CreateAContainer/17-folder-list.png

The contents of a folder may be viewed by clicking the folder listing.

.. image:: /assets/howtos/CreateAContainer/18-folder-contents.png


.. _swift_create_deep_folder:

Create Deep Folders
-------------------

When viewing a folder, a deep folder can be created by selecting `+ Folder`, as :ref:`previously <swift_create_folder>` when viewing a container.

.. image:: /assets/howtos/CreateAContainer/19-subfolder-create.png

Multiple levels of folders may be created by including delimiters ('/') between each folder name. Any folders specified that do not exist will be created.

.. image:: /assets/howtos/CreateAContainer/20-subfolders-create.png

This can equivalently be done when viewing the container, outside all folders.

.. image:: /assets/howtos/CreateAContainer/21-subfolders-create-alt.png

Alternatively, the new folder names may be included when uploading a file.

.. image:: /assets/howtos/CreateAContainer/22-subfolders-create-alt-2.png

As when viewing a container, the next level of folders will be listed alongside files in the folder you are viewing. Deeper folders and files will not be displayed without navigating into their parent folders by selecting them.

.. image:: /assets/howtos/CreateAContainer/23-subfolders-contents.png


.. _swift_delete_object:

Delete a File
-------------

To delete an individual file, select the arrow next to `Download` to view the file options, then clicking `Delete`.

.. image:: /assets/howtos/CreateAContainer/11-object-options.png

On the `Delete Files` screen, select `Delete` to begin deletion.

.. image:: /assets/howtos/CreateAContainer/24-delete-file.png

If successful, a progress bar should fill, and the `Delete Files` screen can be dismissed by clicking `OK` or `X`.


.. _swift_delete_folder:

Delete a Folder
---------------

To delete a folder and its contents, click the `Delete` button.

.. image:: /assets/howtos/CreateAContainer/25-delete-folder.png

As for a file, on the `Delete Files` screen, select `Delete` to begin deletion.

.. image:: /assets/howtos/CreateAContainer/26-delete-folder-final.png

If successful, a progress bar should fill, and the `Delete Files` screen can be dismissed by clicking `OK` or `X`.


.. _swift_delete_multiple_objects:

Delete Multiple Files or Folders
--------------------------------

Select the files and/or folders to be deleted by clicking the checkbox next to each. All file and folder in the current folder or container can be selected by clicking the checkbox next to `Name`.

.. image:: /assets/howtos/CreateAContainer/27-files-select.png

Click the `Delete` icon next to `+ Folder`.

.. image:: /assets/howtos/CreateAContainer/28-files-delete.png

As for individual files and folders, select `Delete` on the the `Delete Files` screen to begin deletion. You can then dismiss the `Delete Files` screen.


.. _swift_delete_containers:

Delete a Container
------------------

In order to delete a container, all files and folders in the container must first be deleted. Then, with the container selected, click the `Delete` icon next to its name.

.. image:: /assets/howtos/CreateAContainer/29-container-delete.png

On the the `Confirm Delete` screen, select `Delete` to begin deletion.

.. image:: /assets/howtos/CreateAContainer/30-container-delete-final.png

If successful, a success message will appear, and the container will no longer be listed.


.. _swift_edit_objects:

Edit a File
-----------

To edit a file, select the arrow next to `Download` to view the options, then click `Edit`.

.. image:: /assets/howtos/CreateAContainer/11-object-options.png

On the `Edit File` screen, select `Choose file` to specifiy the new contents of the file, then click `Edit File`.

.. image:: /assets/howtos/CreateAContainer/31-object-edit.png

If successful, a success message will appear, and the file details will be updated.

.. warning::

  Editing a file will remove any existing properties, apart from ``Orig-Filename``, which will be set to the uploaded file name.
  See also: :ref:`swift_cli_editing_objects`

.. _swift_copy_objects:

Copy a File
-----------

To copy a file, select the arrow next to `Download` to view the options, then click `Copy`.

.. image:: /assets/howtos/CreateAContainer/11-object-options.png

On the `Copy Object` screen, enter the name of the destination container and destination file name, then click `Copy Object`.

.. image:: /assets/howtos/CreateAContainer/32-object-copy.png

If successful, a success message will appear, and the copied file will be listed.


.. _swift_terms:

Explanation of terms
--------------------

Object:
An object is the basic storage entity and any optional metadata that represents the data you store. When you upload data, the data is stored as-is (with no compression or encryption).

Container:
A container is a storage compartment for your data and provides a way for you to organize your data. Containers can be visualised as directories in a Linux system. However, unlike directories, containers cannot be nested. Data must be stored in a container and hence the objects are created within a container.

Folder/Pseudo-folder:
Similar to folders in your desktop operating system. They are virtual collections defined by a common prefix on the object’s name.

Object Store:
Object Store provides a system for data storage that enables users to access the same data, both as an object and as a file, thus simplifying management and controlling storage costs.


References
----------

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/chap-managing_object_store

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/components_of_object_storage

https://docs.openstack.org/horizon/train/user/manage-containers.html