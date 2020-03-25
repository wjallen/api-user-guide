Share Data with Others
======================

The Tapis CLI makes it possible to share data with other users on the same
tenant. Use your best judgement in deciding whether to copy shared data or link
against shared data with the understanding that storage space is limited. The
guide below demonstrates how to modify permissions on a given data set to share
it in place without copying.


Find Another User
-----------------

To share files with another user on the same tenant, you must first know their
username. The Tapis CLI has a set of tools that can be used to find other users.
View your own user profile by issuing:

.. code-block:: bash

   $ tapis profiles show self
   +--------------+------------------------+
   | Field        | Value                  |
   +--------------+------------------------+
   | first_name   | William                |
   | last_name    | Allen                  |
   | email        | wallen@tacc.utexas.edu |
   | mobile_phone |                        |
   | phone        |                        |
   | username     | wallen                 |
   +--------------+------------------------+


Each of the fields stored in the user profile is queryable using the ``tapis
profiles search`` command. Some more common examples include:

.. code-block:: bash

   # Search for another user by first name
   $ tapis profiles search --first-name eq William

   # Search for another user by last name
   $ tapis profiles search --last-name eq Allen

   # Search for another user by email address
   $ tapis profiles search --email eq wallen@tacc.utexas.edu


Once you have identified the correct username, you can query it to make sure it
is the person you are looking for:

.. code-block:: bash

   $ tapis profiles show jdoe
   +--------------+----------------------+
   | Field        | Value                |
   +--------------+----------------------+
   | first_name   | John                 |
   | last_name    | Doe                  |
   | email        | jdoe@tacc.utexas.edu |
   | mobile_phone |                      |
   | phone        |                      |
   | username     | jdoe                 |
   +--------------+----------------------+


Share Files with Another User
-----------------------------

File permissions are managed similar to Unix file permissions. To list the
permissions on an existing file on one of your storage systems, issue:

.. code-block::bash

   $ tapis files pems list agave://utrc-home.wallen/test_folder/local_file.txt
   +----------+------+-------+---------+
   | username | read | write | execute |
   +----------+------+-------+---------+
   | wallen   | True | True  | True    |
   | wma_prtl | True | True  | True    |
   +----------+------+-------+---------+

To add permissions for another user (with username ``jdoe``) to read the file,
use the ``tapis files pems grant`` with the following positional arguments:

.. code-block::bash

   $ (tapis-cli-3.7.5) wallen-mbp19:tapis-cli wallen$ tapis files pems grant agave://utrc-home.wallen/test_folder/local_file.txt jfonner READ
   +----------+------+-------+---------+
   | username | read | write | execute |
   +----------+------+-------+---------+
   | jfonner  | True | False | False   |
   | wallen   | True | True  | True    |
   | wma_prtl | True | True  | True    |
   +----------+------+-------+---------+

.. warning::

   Recursive permission changes are not implemented yet


Now, a user with username ``jdoe`` has permissions to read the file. Valid
values for setting permission are  ALL, READ, WRITE, READ_WRITE, EXECUTE,
READ_EXECUTE, WRITE_EXECUTE, and NONE. However, before ``jdoe`` can access the
file, they also need permissions on the private storage system. To see who has
access to your storage system, perform:


.. code-block:: bash

   $ tapis systems roles list utrc-home.wallen
   +----------+-------+
   | username | role  |
   +----------+-------+
   | wma_prtl | OWNER |
   | wallen   | OWNER |
   +----------+-------+

To add your collaborator to your system use:

.. code-block:: bash

   $ tapis systems roles grant utrc-home.wallen jdoe GUEST
   +----------+---------+
   | Field    | Value   |
   +----------+---------+
   | username | jdoe    |
   | role     | GUEST   |
   +----------+---------+

   $ tapis systems roles list utrc-home.wallen
   +----------+-------+
   | username | role  |
   +----------+-------+
   | wma_prtl | OWNER |
   | jdoe     | GUEST |
   | wallen   | OWNER |
   +----------+-------+

Now, a user with username ``jdoe`` can see files with the appropriate
permissions on your storage system. Valid values for setting a role include
GUEST, USER, PUBLISHER, ADMIN, and OWNER.

Finally, ask your collaborator to download the file with the exact same command
you use to download the file:


.. code-block:: bash

   $ tapis files download agave://utrc-home.wallen/test_folder/local_file.txt


Revoke Permissions
------------------

If you want to revoke permissions, make sure to revoke permissions both on the
shared file as well as the storage system:

.. code-block:: bash

   # Revoke permissions on the shared file
   $ tapis files pems revoke agave://utrc-home.wallen/test_folder/local_file.txt jdoe

   # Revoke permissions on the private storage system
   $ tapis systems roles revoke utrc-home.wallen jdoe

You can also blanket revoke permissions from all non-owner users:

.. code-block:: bash

   # Revoke permissions on the shared file for all users
   $ tapis files pems drop agave://utrc-home.wallen/test_folder/local_file.txt

   # Revoke permissions on the private storage system for all users
   $ tapis systems roles drop utrc-home.wallen


Share Files Using Postits
-------------------------

Another convenient way to share data is the Tapis postits service. Postits
generate a short URL with a user-specified lifetime and limited number of uses.
Anyone with the URL can paste it into a web browser, or curl against it on the
command line. To create a postit:


.. code-block:: bash

   $ tapis postits create -L 3600 -m 5 https://api.tacc.utexas.edu/files/v2/media/system/utrc-home.wallen/test_folder/file.txt
   +---------------+-----------------------------------------------------------------------------------------+
   | Field         | Value                                                                                   |
   +---------------+-----------------------------------------------------------------------------------------+
   | postit        | 843f372264d653ecea967f790ca90381                                                        |
   | remainingUses | 5                                                                                       |
   | expires       | 2020-03-25T15:06:56-05:00                                                               |
   | url           | https://api.tacc.utexas.edu/files/v2/media/system/utrc-home.wallen/test_folder/file.txt |
   | creator       | wallen                                                                                  |
   | created       | 2020-03-25T14:06:56-05:00                                                               |
   | noauth        | False                                                                                   |
   | method        | GET                                                                                     |
   | postit_url    | https://api.tacc.utexas.edu//postits/v2/843f372264d653ecea967f790ca90381                |
   +---------------+-----------------------------------------------------------------------------------------+


The response from this command includes a URL which can be pasted into a web
browser or curled on the command line:

.. code-block:: bash

   https://api.tacc.utexas.edu//postits/v2/843f372264d653ecea967f790ca90381


This postit will work for 5 downloads (``-m 5``) and only available for one hour
(3600 seconds, ``-L 3600``). The creator of the postit can list and delete their
postits with the following commands:

.. code-block:: bash

   $ tapis postits list
   +----------------------------------+---------------+---------------------------+-----------------------------------------------------------------------------------------+
   | postit                           | remainingUses | expires                   | url                                                                                     |
   +----------------------------------+---------------+---------------------------+-----------------------------------------------------------------------------------------+
   | 843f372264d653ecea967f790ca90381 |             5 | 2020-03-25T15:06:56-05:00 | https://api.tacc.utexas.edu/files/v2/media/system/utrc-home.wallen/test_folder/file.txt |
   +----------------------------------+---------------+---------------------------+-----------------------------------------------------------------------------------------+

   $ tapis postits delete 843f372264d653ecea967f790ca90381
