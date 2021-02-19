Harbor Private Docker Registry
##############################

.. contents::

Background
==========

The STFC Cloud team runs a Harbor private registry at https://harbor.stfc.ac.uk

This service allows users to upload docker images, and manage their lifecycles internally. This has a number of advantages over using a public repository:

- Login uses IRIS, so existing credentials can be used rather than creating new accounts and teams on external providers.
- Images aren't subjected to free / paid account restrictions on other platforms.
- Built in vulnerability scanner indicates any CVEs applicable to the image on upload and a periodic basis.
- Projects can be made public, or privately available to a specific group of users.

Getting Started
===============

.. _harbor_sign_up:

Signing Up
----------

Anybody with an IRIS account is able to sign up. An IRIS account can be applied for using facility credentials `here. <https://iris-iam.stfc.ac.uk/login>`_

To associate your IRIS account simply go to `Harbor <https://harbor.stfc.ac.uk>`_ and click **Login Via OIDC Provider**. If it's your first sign in, you will be asked to confirm basic details being shared with Harbor. Harbor will then confirm the account details (which cannot be modified).

.. _harbor_sign_in:

Signing In
----------

If you have already signed up (as described in :ref:`harbor_sign_up`) you can login by simply clicking **Login Via OIDC Provider**.

Entering your facility/IRIS credentials into the Harbor login page will not work, as IRIS IAM handles authentication on our behalf.

Creating a Project
------------------

Please :ref:`contact us <contact_email>` to request a new project, please include the following details in your email:

- The project name (all lowercase, can use - and _)
- Brief description of planned usage
- Is the project public or private (see :ref:`private_repo_details`)
- Estimated storage requirements (GB)
- Planned User / Group Permissions (see :ref:`harbor_perms`)
- (Optional) Retention Rules (see :ref:`harbor_retention`)

Using Harbor
============

.. _harbor_perms:

Project Permissions
-------------------

Permissions are presented in increasing scope, each level presented includes new permissions and all permissions from the previous level:

- Anonymous
    A user who has not logged into Harbor using their Docker client. They can only pull from public repositories.
- Limited Guest
    A user who can pull from private and public projects, but cannot see logs or members of a project. Useful for end-users and machines.
- Guest
    A user who can pull images from a private project and see other members of the project.
- Developer
    A user who can pull and push images to the project. They cannot delete an image once pushed.
- Project Master
    A user who can also trigger manual security scans and delete images.
- Project Admin
    This person can manage all aspects of a named project, including changing user and group permissions.
- Harbor Administrators
    Can create and delete projects. Can update a project's admin to be a different user as required upon support request.

.. _private_repo_details:

Private Projects
----------------

.. Important:: **Secrets in Images**

    A private project/repository does not mean users should include secrets into their Docker images. Please keep secrets separate to images by passing them through .env files or environment flags. A good rule-of-thumb is asking, "if this image ever leaked could a system become compromised from the details within".

Users can request a private repository; the names, images and associated members of these projects are hidden from non-members.  

For most use-cases a public project is preferred:

- Images are immutable; a SHA reference cannot be changed.
- Anonymous pulls removes the requirements on securely distributing and storing access tokens.
- Users can start software with a single docker command, lowering the barrier of entry for deployment. 

Some examples where a private projects should be considered are:

- When software licenses are required per container instance
- Mirroring / storing proprietary software (check License Agreement beforehand)
- Confidential or unannounced/internal development projects
- Where scientific data is included but subject to access restrictions

Machines will not be able to pull from a private repository without first :ref:`logging in <login_harbor>`.

.. _login_harbor:

Authenticating Docker
---------------------

.. warning::

    `A credentials store is highly recommended. <https://docs.docker.com/engine/reference/commandline/login/#credentials-store>`_ On machines without a credentials store your token is stored in plain-text within your user profile.

Logging in grants you the ability to pull and push to projects where you have appropriate permissions:

- :ref:`Sign into Harbor <harbor_sign_in>`
- Take note of your profile name in the top-right
- Click on the profile name and click **User Profile**
- Copy the CLI secret can be copied using the copy action
- On the target machine run 

.. code:: console

    docker login -u <profile_name> https://harbor.stfc.ac.uk

- It will prompt you for your access token, paste in the previously copied token
- Docker will return if the login was a success and persist this between reboots


Rotating Access Token
---------------------

This is useful if your Docker token has been, or is possibly compromised, or on a machine you no longer have access to. Rotating keys does **not** flag or log your account in any way, so please feel free to use this proactively.

Rotating the access token will generate a new token whilst invalidating the old token and is simple:

- :ref:`Sign into Harbor <harbor_sign_in>`
- Click on the profile name and click **User Profile**
- Click the 3 dots next to **CLI secret**
- Select **Generate Secret**
- Confirm you are happy to discard your old token
- On each machine you require access :ref:`re-login <login_harbor>`

Tagging and Pushing Images
--------------------------

Images should include the name of the harbor server, or they will implicitly use Docker Hub:

.. code:: console

    # For tagging as part of the build
    docker image build . -t harbor.stfc.ac.uk/<project_name>/<image_name>:<tag>

    # For re-tagging an existing image
    docker tag <old_tag> harbor.stfc.ac.uk/<project_name>/<image_name>:<tag>

Here is a worked example using the image `Ubuntu`, on the `latest` tag to a project called `harbor_example`

.. code:: console

    # Build a new Ubuntu image
    docker image build ubuntu -t harbor.stfc.ac.uk/harbor_example/ubuntu:latest

    # For re-tagging an existing Ubuntu image
    docker tag ubuntu/ubuntu:latest harbor.stfc.ac.uk/harbor_example/ubuntu:latest


To push an image to the repository the following command can be used:

.. code:: console

    docker push harbor.stfc.ac.uk/<project_name>/<image_name>:<tag>

For example to mirror the image `ubuntu:latest` from Docker Hub into Harbor Project `my_project`:

.. code:: console

    # This assumes the tag step above was completed
    docker image push harbor.stfc.ac.uk/my_project/ubuntu:latest

.. _harbor_retention:

Advanced: Project Retention Rules
---------------------------------

**Requests**

Up to 15 retention rules can be set on a per-project basis.

Harbor will consider all repositories and all tags eligible for deletion after a user specified number of day **or** after a number of artifacts.

We can also white-list or black-list tag patterns or repository names that are subject to auto-retention rules.

For example, in a project with repositories `foo, bar and baz` we can specify only `foo, baz` to be auto collected after 60 days, whilst `bar` will only delete tags with `*beta*` in their name after 20 days.

If your putting in a support request for retention rules please describe the above in a request, we will configure the rules per your description. For users manually confusing their rules an additional reference follows.

**Manual Config**

Harbor currently has limited regex capabilities for expressing rules. By default the repository list and tags are set to everything `**`.

To specify a list of items, for example `foo bar baz` replace `**` with `{foo,bar,baz}` which will match all.

Care must be taken with semantic versioning. Unlike regex a * character will only match a single character input, for example a retention rule for `v*.*-beta` will match `v1.1-beta` and `v9.5-beta` but not `v10.1-beta`. Full regex support is currently in the feature-request stage upstream.

The rules for a project can be configured by:

- Navigating to the project
- Select the Policy tag
- Ensure Tag Retention is selected
- Configure the rules and schedule as required
