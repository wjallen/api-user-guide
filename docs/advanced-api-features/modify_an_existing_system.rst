Modify an Existing System
=========================

Tapis systems are described by ``json`` files and can be added to the tenant
using the command line interface. To modify a system after it has been added,
you must edit the original ``json`` file and use the CLI to submit the change.

Modify a Storage System
-----------------------

In a
`previous section <create_a_private_system.html>`_
of this user guide, we registered a new storage system called
``tacc.work.wallen`` using the following json, which was stored in a file called
``tacc.work.wallen.json``:

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

If you need to change the hostname, paths, ssh keys, or any other field (other
than the ``id``, which is immutable), the appropriate method would be to edit
the above file to reflect the change, then use the Tapis CLI to edit the
existing storage system. For the purposes of this example, we may want to change
the plain text ``name`` parameter to include more detail. Modify the json file
and submit the changes as follows:

.. code-block:: bash

   $ tapis systems update -F tacc.work.wallen.json tacc.work.wallen
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
   | name                 | Storage system for TACC work directory via Stampede2 |
   | owner                | wallen                                               |
   | public               | False                                                |
   | revision             | 2                                                    |
   | scheduler            | None                                                 |
   | scratchDir           | None                                                 |
   | site                 | None                                                 |
   | status               | UP                                                   |
   | type                 | STORAGE                                              |
   | uuid                 | 7043710487649971734-242ac113-0001-006                |
   | workDir              | None                                                 |
   +----------------------+------------------------------------------------------+

The plain text response should include the new value for the ``name`` parameter.
You can also use the ``tapis systems history`` command to check that the update
was accepted:

.. code-block:: bash

   $ tapis systems history tacc.work.wallen
   +---------+----------------------+-------------------------+
   | status  | created              | description             |
   +---------+----------------------+-------------------------+
   | CREATED | 2020-03-31T01:26:17Z | This system was created |
   | UPDATED | 2020-03-31T02:43:40Z | This system was updated |
   +---------+----------------------+-------------------------+


Modify an Execution System
--------------------------

In a
`previous section <create_a_private_system.html>`_
we registered a new execution system for the Stampede2 HPC cluster. In our
system description, we only included one queue (``normal``), although Stampede2
has many more queues available. To add an additional queue, return to the
original json file called ``tacc.stampede2.wallen.json`` and add another json
object to the queue array:

.. code-block::
   :emphasize-lines: 12-20

   ...
   "queues": [
     {
       "name": "normal",
       "maxProcessorsPerNode": 68,
       "maxMemoryPerNode": "96GB",
       "maxNodes": 256,
       "maxRequestedTime": "48:00:00",
       "customDirectives": "-A <enter allocation name here>",
       "default": true
     },
     {
       "name": "skx-normal",
       "maxProcessorsPerNode": 48,
       "maxMemoryPerNode": "192GB",
       "maxNodes": 128,
       "maxRequestedTime": "48:00:00",
       "customDirectives": "-A <enter allocation name here>",
       "default": true
     }
   ]
   ...

Save that new file and update the existing system with the following:

.. code-block:: bash

   $ tapis systems update -F tacc.stampede2.wallen.json tacc.stampede2.wallen
   +----------------------+---------------------------------------+
   | Field                | Value                                 |
   +----------------------+---------------------------------------+
   | available            | True                                  |
   | default              | False                                 |
   | description          | Execution system for TACC Stampede2   |
   | executionType        | HPC                                   |
   | globalDefault        | False                                 |
   | id                   | tacc.stampede2.wallen                 |
   | lastModified         | just now                              |
   | maxSystemJobs        | 2147483647                            |
   | maxSystemJobsPerUser | 50                                    |
   | name                 | Execution system for TACC Stampede2   |
   | owner                | wallen                                |
   | public               | False                                 |
   | revision             | 2                                     |
   | scheduler            | SLURM                                 |
   | scratchDir           | /scratch/03439/wallen/                |
   | site                 | None                                  |
   | status               | UP                                    |
   | type                 | EXECUTION                             |
   | uuid                 | 5042654862881657322-242ac113-0001-006 |
   | workDir              |                                       |
   +----------------------+---------------------------------------+

   $ tapis systems queues list tacc.stampede2.wallen
   +------------+-------------+---------+-------------+------------------+
   | name       | description | default | maxUserJobs | maxRequestedTime |
   +------------+-------------+---------+-------------+------------------+
   | skx-normal | None        | True    |          -1 | 48:00:00         |
   | normal     | None        | False   |          -1 | 48:00:00         |
   +------------+-------------+---------+-------------+------------------+
