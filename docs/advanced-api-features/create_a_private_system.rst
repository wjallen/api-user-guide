Create a Private System
=======================

Many tenants automatically provide their users with private storage and
execution systems connecting to TACC resources, but some do not. This guide
walks through the process of creating your own private systems. This can also
be used to help you connect to non-TACC lab servers, cloud VMs, and clusters.


Gather Relevant Information
---------------------------

To register any system in Tapis, you need the hostname and login credentials.
The preferred login credentials are username and SSH key pairs. Storage systems
(for managing files) require a default path where you have write access.
Execution systems (for running jobs) also require a default path where job
runtime files will be staged and the job will be executed. If it is an 'HPC'
type execution system, then you also need information about the queueing system
(queue names, limits, etc.).

For this demonstration, we will set up a storage system to access the TACC work
directory (via Stampede2):

.. code-block:: bash

    hostname: stampede2.tacc.utexas.edu
    username: wallen
    credentials: <ssh keys>
    storage path: /work/03439/wallen


And we will set up an execution system for the Stampede2 HPC cluster:

.. code-block:: bash

   hostname: stampede2.tacc.utexas.edu
   username: wallen
   credentials: <ssh keys preferred>
   storage path: /work/03439/wallen
   job runtime path: /scratch/03439/wallen
   queue_type: SLURM
   queue: normal (limits in Stampede2 user guide)


.. note::

   This guide assumes you have the appropriate permissions and credentials to
   access Stampede2. These can be attained by having an active
   `Stampede2 Allocation <https://portal.tacc.utexas.edu/allocations-overview>`_


Register a Storage System
-------------------------

To register a system, you need to assemble a json description of the above
requirements and some other metadata. Start by saving the following storage
system template in a file called ``tacc.work.wallen.json``:

.. code-block:: json

   {
     "id": "tacc.work.wallen",
     "name": "Storage system for TACC work directory",
     "description": "Storage system for TACC work directory via Stampede2",
     "type": "STORAGE",
     "storage": {
       "host": "stampede2.tacc.utexas.edu",
       "port": 22,
       "protocol": "SFTP",
       "rootDir": "/work/03439/wallen",
       "homeDir": "/",
       "auth":{
         "username":"wallen",
         "publicKey": " <enter public key here> ",
         "privateKey": " <enter private key here> ",
         "type": "SSHKEYS"
       }
     }
   }

Most fields are fairly self explanatory, but here is a brief breakdown of the
important options:

  * ``id``: a unique identifier and how the system name appears for ``tapis systems list`` commands
  * ``name``: common display name for the system
  * ``description``: optional long plain text description of the system
  * ``type``: can be ``STORAGE`` or ``EXECUTION``
  * ``host``: IP address or hostname of system
  * ``rootDir``: path of the virtual root directory on the remote system
  * ``homeDir``: path relative to ``rootDir`` for ``tapis files -`` operations
  * ``username``: this is your username for the target system
  * ``publicKey``: cut and paste your public key here
  * ``privateKey``: cut and paste your private key here


Edit the username and paths in the above template to match your username and
work folder. A copy of your public key should be added to the authorized_keys
files on the remote host. The public and private key should be pasted on one
line each (wrapping around) similar to the following:

.. code-block:: json

   {
     "auth":{
       "username": "username",
       "publicKey": "ssh-rsa AAAAB3NzaC1yc2EBBAADAQABMQRgQChJ6bzejqSuJdTi+VwMif8qouSSlYwrVt0EWVduKZHpzOnS1zlknAyYXmQQFcaJ+vNAQayVMTqv+A+1lzxppTdgZ0Dn42EOYWRa6B/IEMPzDuKb7F0qNFiH9m+OZJDYdIWS1rlN1oK32jHUi0xV8kM3KOLf2TIjDBUyZRpMGyQ= user@email.com",
       "privateKey": "-----BEGIN RSA PRIVATE KEY-----nMIVCXAIBAAKBgQRhJ6bzejqSuJdTi+VwMif8qoyuSSlYwrVt0EWVdkFvA+wmxlOcnLMJOYotSyu0JqY/TeW6reNBMkTkVU8FgXJ2k+4agNrphxKCWmQbC4Xm+CW5N6HiIBZo/TxzDaAmsNGklmVfZGO+8cCDqdKIlF0hqxytI8GgtiHImg2j+nwcIQT3ojER45I+6hYLj95HnSyyC7rEtjIBCvW8FVmT7JCDnS0BwAkmnRt0NPzrliEk1k+swkCTp3SOHSk4SsJPuLcC7OW6pkjD6AyHV4ZrYy0US/Z+Zmn01Lhgw0sNjQL8PyJuVeFysp9Sr40c77OYbVGbOAJGKGtYsD6x3 0Cvz+vqQ0VpQPCOiMf2tytglUNBkiEVThkm+Nl36yxRmcGCLEh9EGTWNuD++ZT+nka6MvIN2NSsXJD32sw15g4A0bmzSXnbfFg8TBAjGTDW7l0P8prFrtQ8Wml14390b29l1ptAyE=n-----END RSA PRIVATE KEY-----",
       "type": "SSHKEYS"
     }
   }

.. warning::

   Remember, never share this json file because it contains a plain text copy of
   your private key.

You will need to keep a copy of this file to edit the storage system in the
future. To register this system with Tapis, use the following command:

.. code-block:: bash

   $ tapis systems create -F tacc.work.wallen.json
   +----------------------+------------------------------------------------------+
   | Field                | Value                                                |
   +----------------------+------------------------------------------------------+
   | available            | True                                                 |
   | default              | False                                                |
   | description          | Storage system for TACC work directory via Stampede2 |
   | executionType        | None                                                 |
   | globalDefault        | False                                                |
   | id                   | tacc.work.wallen                                     |
   | lastModified         | just now                                             |
   | maxSystemJobs        | None                                                 |
   | maxSystemJobsPerUser | None                                                 |
   | name                 | Storage system for TACC work directory               |
   | owner                | wallen                                               |
   | public               | False                                                |
   | revision             | 1                                                    |
   | scheduler            | None                                                 |
   | scratchDir           | None                                                 |
   | site                 | None                                                 |
   | status               | UP                                                   |
   | type                 | STORAGE                                              |
   | uuid                 | 7043710487649971734-242ac113-0001-006                |
   | workDir              | None                                                 |
   +----------------------+------------------------------------------------------+


Confirm that it worked by searching for the storage system and listing files
in the root directory:


.. code-block:: bash

   $ tapis systems search --id eq tacc.work.wallen
   +------------------+----------------------------------------+--------+---------+
   | id               | name                                   | status | type    |
   +------------------+----------------------------------------+--------+---------+
   | tacc.work.wallen | Storage system for TACC work directory | UP     | STORAGE |
   +------------------+----------------------------------------+--------+---------+

   $ tapis files list agave://tacc.work.wallen/
   +-------------------+--------------+--------+
   | name              | lastModified | length |
   +-------------------+--------------+--------+
   | archive           | 5 months ago |   4096 |
   | class-software    | 2 months ago |   4096 |
   | corral            | 2 weeks ago  |     13 |
   | cyverse           | a year ago   |   4096 |
   | files             | 2 months ago |  12288 |
   | frontera          | 2 months ago |   4096 |
   | hikari            | 3 years ago  |   4096 |
   | jetstream         | 2 years ago  |   4096 |
   | jobs              | 6 months ago |   4096 |
   | jupyter           | 4 months ago |   4096 |
   | lonestar          | 5 months ago |   4096 |
   | maverick          | a year ago   |   4096 |
   | maverick2         | 3 weeks ago  |   4096 |
   | public            | 3 days ago   |   4096 |
   | rpmbuild          | a year ago   |   4096 |
   | sd2e              | 8 months ago |   4096 |
   | share-files       | 3 months ago |   4096 |
   | singularity_cache | a month ago  |   4096 |
   | stampede          | 3 years ago  |   4096 |
   | stampede2         | 3 weeks ago  |   4096 |
   | test_folder       | 2 days ago   |   4096 |
   | wallen            | 2 months ago |   4096 |
   | wrangler          | 2 months ago |   4096 |
   +-------------------+--------------+--------+


Register an Execution System
----------------------------

An execution system contains many of the same fields as a storage system, but it
is a bit more involved. Save the following template for a Stampede2 execution
system into a file called ``tacc.stampede2.wallen``:


.. code-block:: json

   {
     "id": "tacc.stampede2.wallen",
     "name": "Execution system for TACC Stampede2",
     "description": "Execution system for TACC Stampede2",
     "type": "EXECUTION",
     "executionType": "HPC",
     "scheduler": "SLURM",
     "maxSystemJobsPerUser": 50,
     "scratchDir": "/scratch/03439/wallen",
     "login": {
       "host": "stampede2.tacc.utexas.edu",
       "port": 22,
       "protocol": "SSH",
       "auth": {
         "username": "wallen",
         "publicKey": " <enter public key here> ",
         "privateKey": " <enter private key here> ",
         "type": "SSHKEYS"
       }
     },
     "storage": {
       "host": "stampede2.tacc.utexas.edu",
       "port": 22,
       "protocol": "SFTP",
       "rootDir": "/",
       "homeDir": "/work/03439/wallen",
       "auth": {
         "username": "wallen",
         "publicKey": " <enter public key here> ",
         "privateKey": " <enter private key here> ",
         "type": "SSHKEYS"
       }
     },
     "queues": [
       {
         "name": "normal",
         "maxProcessorsPerNode": 68,
         "maxMemoryPerNode": "96GB",
         "maxNodes": 256,
         "maxRequestedTime": "48:00:00",
         "customDirectives": "-A <enter allocation name here>",
         "default": true
       }
     ]
   }

Some of the new or changed fields in this execution system include:

  * ``type``: execution system rather than storage system
  * ``executionType``: ``HPC`` indicates a cluster with a job scheduler
  * ``scheduler``: Stampede2 uses a SLURM scheduler
  * ``maxSystemJobsPerUser``: maximum concurrent jobs on the system per user
  * ``scratchDir``: path for job working directory at runtime, relative to ``rootDir``
  * ``login``: similar to `storage`, host and credential information
  * ``queues``: an array of batch queue definitions for the HPC system

For this execution system, there are two locations to cut and paste your SSH
keys. Again, because keys will be stored in plain text in this file, do not
share this file with anyone and keep it secure. In addition, the ``queues``
parameter has an option called ``customDirectives`` which should contain the
name of an allocation you have access to on Stampede2. And finally, as before,
make sure to change the username and paths to match your account on the HPC
system.

Once the appropiate changes have been made to the json file, register the system
with Tapis using the following command:

.. code-block:: bash

   $ tapis systems create -F tacc.stampede2.wallen.json
   +----------------------+--------------------------------------+
   | Field                | Value                                |
   +----------------------+--------------------------------------+
   | available            | True                                 |
   | default              | False                                |
   | description          | Execution system for TACC Stampede2  |
   | executionType        | HPC                                  |
   | globalDefault        | False                                |
   | id                   | tacc.stampede2.wallen                |
   | lastModified         | just now                             |
   | maxSystemJobs        | 2147483647                           |
   | maxSystemJobsPerUser | 50                                   |
   | name                 | Execution system for TACC Stampede2  |
   | owner                | wallen                               |
   | public               | False                                |
   | revision             | 1                                    |
   | scheduler            | SLURM                                |
   | scratchDir           | /scratch/03439/wallen/               |
   | site                 | None                                 |
   | status               | UP                                   |
   | type                 | EXECUTION                            |
   | uuid                 | 807903008371577322-242ac113-0001-006 |
   | workDir              |                                      |
   +----------------------+--------------------------------------+


Finally, confirm that the system exists by searching for it:

.. code-block:: bash

   $ tapis systems search --public eq false
   +----------------------------------+----------------------------------------+--------+-----------+
   | id                               | name                                   | status | type      |
   +----------------------------------+----------------------------------------+--------+-----------+
   | tacc.stampede2.wallen            | Execution system for TACC Stampede2    | UP     | EXECUTION |
   | tacc.work.wallen                 | Storage system for TACC work directory | UP     | STORAGE   |
   +----------------------------------+----------------------------------------+--------+-----------+


Additional Help
---------------

Further information about creating storage and execution systems, including full
descriptions of the parameters above as well as other optional parameters, can
be found in the
`Tapis Documentation <https://tacc-cloud.readthedocs.io/en/latest/>`_
