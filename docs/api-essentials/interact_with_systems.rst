Interact with Systems
=====================

A Tapis *system* is a server or collection of servers associated with a single
hostname. They may be public or private, and they may either be **storage**
systems (used for storing files) or **execution** systems (used for running
jobs).

On some tenants, users are provide private storage and execution systems
attached to TACC resources. If you would like to create your own systems
attached to different resources, skip ahead to
`Create a Private System <create_a_private_system.html>`_.


Find Systems
------------

Systems can be discovered using the ``tapis systems list`` command:

.. code-block::

   $ tapis systems list
   +----------------------------------+-------------------------------------------------+--------+-----------+
   | id                               | name                                            | status | type      |
   +----------------------------------+-------------------------------------------------+--------+-----------+
   | tapis.storage.system             | Tapis storage system                            | UP     | STORAGE   |
   | tapis.execution.system           | Tapis execution system                          | UP     | EXECUTION |
   | docking.storage                  | Storage VM for the drug discovery portal        | UP     | STORAGE   |
   | utrc-home.wallen                 | utrc-home.wallen                                | UP     | STORAGE   |
   | polar.home.wallen                | polar.home.wallen                               | UP     | STORAGE   |
   | jetstream-storage-wallen         | Jetstream storage system                        | UP     | STORAGE   |
   | csc.2018.storage                 | Storage system for CSC 2018 Institute           | UP     | STORAGE   |
   | docking.exec.ls5                 | Lonestar 5 Supercomputer                        | UP     | EXECUTION |
   | docking.slurm.stampede           | Docking's Stampede SLURM Execution Host         | UP     | EXECUTION |
   | docking.work.storage.stampede    | Stampede $WORK storage                          | UP     | STORAGE   |
   | docking.fork.fleming             | Fork Execution VM for the drug discovery portal | UP     | EXECUTION |
   | docking.storage2                 | Storage VM for the drug discovery portal        | UP     | STORAGE   |
   | docking.exec.lonestar            | Lonestar 4 Supercomputer                        | UP     | EXECUTION |
   +----------------------------------+-------------------------------------------------+--------+-----------+


On this tenant, the user sees numerous storage and execution systems. All
systems are either public or private (with varying degrees of privacy), each
of which are shown here.

On tenants with many more systems, it may be useful to narrow the list with
the ``tapis systems search`` command. For example, to discover systems only
owned by you:

.. code-block:: bash

   $ tapis systems search --owner eq myusername
   +--------------------------------------+--------------------------+--------+-----------+
   | id                                   | name                     | status | type      |
   +--------------------------------------+--------------------------+--------+-----------+
   | jetstream-storage-myusername         | Jetstream storage system | UP     | STORAGE   |
   | docking-exec-lonestar5-myusername    | Lonestar5 Supercomputer  | UP     | EXECUTION |
   | docking-storage-lonestar5-myusername | Lonestar5 Supercomputer  | UP     | STORAGE   |
   +--------------------------------------+--------------------------+--------+-----------+


Other useful searches might include:

.. code-block:: bash

   $ tapis systems search --owner neq myusername    # systems you don't own
   $ tapis systems search --public eq true          # only public systems
   $ tapis systems search --public eq false         # only private systems
   $ tapis systems search --type eq EXECUTION       # only execution systems
   $ tapis systems search --type eq STORAGE         # only storage systems
   $ tapis systems search --default eq true         # your default system


You can also chain multiple search parameters. For example, to display only
private storage systems:

.. code-block:: bash

   $ tapis systems search --type eq STORAGE --public eq false
   +----------------------------------+--------------------------+--------+---------+
   | id                               | name                     | status | type    |
   +----------------------------------+--------------------------+--------+---------+
   | utrc-home.wallen                 | utrc-home.wallen         | UP     | STORAGE |
   | polar.home.wallen                | polar.home.wallen        | UP     | STORAGE |
   | jetstream-storage-wallen         | Jetstream storage system | UP     | STORAGE |
   | docking-storage-lonestar5-wallen | Lonestar5 Supercomputer  | UP     | STORAGE |
   +----------------------------------+--------------------------+--------+---------+


.. note::

   Don't forget to replace instances of *myusername* with your actual username
   on the tenant



Discover System Hostname and Default Paths
------------------------------------------

To see additional system information, including hostname, root folder locations,
and availability, use the ``tapis systems show`` command with a system ID. For
example, find out more information about your personal storage system as
follows:



.. code-block:: bash

   $ tapis systems show -f json utrc-home.wallen

.. code-block:: json
   :linenos:
   :emphasize-lines: 31,32

   {
     "available": true,
     "default": false,
     "description": "Home system for user: wallen",
     "environment": null,
     "executionType": null,
     "globalDefault": false,
     "id": "utrc-home.wallen",
     "lastModified": "2 years ago",
     "login": null,
     "maxSystemJobs": null,
     "maxSystemJobsPerUser": null,
     "name": "utrc-home.wallen",
     "owner": "wma_prtl",
     "public": false,
     "queues": null,
     "revision": 1,
     "scheduler": null,
     "scratchDir": null,
     "site": "portal.dev",
     "status": "UP",
     "storage": {
       "proxy": null,
       "protocol": "SFTP",
       "mirror": false,
       "port": 22,
       "auth": {
         "type": "SSHKEYS"
       },
       "publicAppsDir": null,
       "host": "data.tacc.utexas.edu",
       "rootDir": "/work/03439/wallen",
       "homeDir": "/"
     },
     "type": "STORAGE",
     "uuid": "4106595520523988504-242ac113-0001-006",
     "workDir": null
   }


The :code:`-f json` flag was provided to render all information about the
system. As described above, this storage system is a gateway to your private
storage space on the TACC `$WORK` filesystem. The `rootDir` is the virtual root
path for operations performed on this system. The highlighted lines emphasize
the host and root directory when performing operations against this system.

In addition to standard host and path information, execution systems also contain
information about queue types and availability.


Check System Status
-------------------

It may be useful to check the status (availability) of a system in a scriptable
way prior to, e.g., uploading files as part of a pipeline. The following command
can be used with extra flags to strip out the useful part of the response:

.. code-block:: bash

   $ tapis systems status -c status -f value
   UP


.. note::

   Use :code:`tapis command subcommand --help` to find usage information for
   each command
