Deploy and Test the App
=======================

At this point, your app bundle directory structure should look similar to:

.. code-block:: text

   fastqc_app/
   ├── Dockerfile                   # Done
   ├── app.json                     # Done
   ├── assets
   │   ├── lib
   │   │   ├── VERSION              # Done
   │   │   └── container_exec.sh    # Do not modify
   │   ├── runner.sh                # Done
   │   └── tester.sh                # Do not modify
   ├── job.json                     # Done
   ├── project.ini                  # Done
   ├── src
   │   └── mycode.py                # Optional
   └── test
       └── SP1.fq                   # Optional



The final steps are to deploy the app in to the catalog on the tenant, then run
a test job to confirm it is working.


Deploy the App
--------------

The ``tapis apps deploy`` command does multiple things. First, it builds and
pushes your docker image to the namespace specified in ``project.ini``. Second,
it uploads the app bundle to a Tapis storage space specified by
``deploymentSystem`` and ``deploymentPath`` in the app json file. Finally, it
adds the app to the tenant.

Navigate in to the app bundle folder and issue:

.. code-block:: bash

   $ pwd
   ~/fastqc_app/

   $ ls
   Dockerfile      app.json        assets/         job.json
   project.ini     src/            test/

   $ tapis apps deploy -W ./
   +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | stage  | message                                                                                                                                                                                     |
   +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | build  | Step 1/4 : FROM tacc/tacc-ubuntu18-mvapich2.3-psm2                                                                                                                                          |
   | build  |  ---> d554e642ddc5                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Step 2/4 : RUN apt-get update     && apt-get upgrade -y     && apt-get install wget -y     && apt-get install zip -y     && apt-get install default-jre -y                                  |
   | build  |  ---> Using cache                                                                                                                                                                           |
   |        |                                                                                                                                                                                             |
   | build  |  ---> aa1f50856b62                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Step 3/4 : RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip     && unzip fastqc_v0.11.9.zip     && rm fastqc_v0.11.9.zip     && chmod +x FastQC/fastqc |
   | build  |  ---> Using cache                                                                                                                                                                           |
   |        |                                                                                                                                                                                             |
   | build  |  ---> 3bb6917b68d6                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Step 4/4 : ENV PATH "/FastQC/:$PATH"                                                                                                                                                        |
   | build  |  ---> Using cache                                                                                                                                                                           |
   |        |                                                                                                                                                                                             |
   | build  |  ---> 356927b0a8f6                                                                                                                                                                          |
   |        |                                                                                                                                                                                             |
   | build  | Successfully built 356927b0a8f6                                                                                                                                                             |
   |        |                                                                                                                                                                                             |
   | build  | Successfully tagged wallen/fastqc_app:0.11.9                                                                                                                                                |
   |        |                                                                                                                                                                                             |
   | push   | The push refers to repository [docker.io/wallen/fastqc_app]                                                                                                                                 |
   | push   | 0.11.9: digest: sha256:29eb2fdb1503fdd38ae311dabfc13958f0910253580614dba0d3ac2dd0753e41 size: 4085                                                                                          |
   | upload | assets/runner.sh                                                                                                                                                                            |
   | upload | assets/tester.sh                                                                                                                                                                            |
   | upload | assets/lib/VERSION                                                                                                                                                                          |
   | upload | assets/lib/container_exec.sh                                                                                                                                                                |
   | create | Created Tapis app fastqc_app-0.11.9 revision 1                                                                                                                                              |
   +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


If all goes well, you should see a successful message at the end of the log
above, and you should see the new app listed in the apps catalog:

.. code-block:: bash

   $ t
   $ tapis apps search --name like fastqc
   +-------------------+----------+------------+------------------+----------+-----------------------+
   | id                | revision | label      | shortDescription | isPublic | executionSystem       |
   +-------------------+----------+------------+------------------+----------+-----------------------+
   | fastqc_app-0.11.9 |        1 | fastqc_app | fastqc app       | False    | tacc.stampede2.wallen |
   +-------------------+----------+------------+------------------+----------+-----------------------+


Submit a Test Job
-----------------

Submitting a test job has been
`described previously <api-essentials/prepare_and_submit_a_job.html>`_
in this how-to guide. Here, testing will be performed in the same way. First,
create an appropriate ``job.json`` file.

.. code-block:: bash

   $ tapis jobs init --no-archive --output fastqc_job.json fastqc_app-0.11.9


Which will output the following json, which can be streamed into a file for
submission:

.. code-block:: json

   {
     "name": "fastqc_app-job-1586180623258",
     "appId": "fastqc_app-0.11.9",
     "batchQueue": "skx-normal",
     "maxRunTime": "01:00:00",
     "memoryPerNode": "1GB",
     "nodeCount": 1,
     "processorsPerNode": 1,
     "archive": false,
     "inputs": {
       "fastq": "agave://tacc.work.wallen/public/SP1.fq"
     },
     "parameters": {},
     "notifications": [
       {
         "event": "FINISHED",
         "persistent": true,
         "url": "wallen@tacc.utexas.edu"
       },
       {
         "event": "FAILED",
         "persistent": true,
         "url": "wallen@tacc.utexas.edu"
       }
     ]
   }

Then, submit the test job:

.. code-block:: bash

   $ tapis jobs submit -F fastqc_job.json
   +--------+------------------------------------------+
   | Field  | Value                                    |
   +--------+------------------------------------------+
   | id     | 0946cc05-f076-4f71-8ef5-3aff0fdf4c71-007 |
   | name   | fastqc_app-job-1586180806899             |
   | status | ACCEPTED                                 |
   +--------+------------------------------------------+


Finally, when the job status is **FINISHED**, inspect and retrieve the output:

.. code-block:: bash

   $ tapis jobs history f0d48348-aea8-46e6-92a8-f18baa33bdea-007

   $ tapis jobs outputs list f0d48348-aea8-46e6-92a8-f18baa33bdea-007

   $ tapis jobs outputs download --progress f0d48348-aea8-46e6-92a8-f18baa33bdea-007


If the file ``SP1.fq.html`` exists, then the run was successful.
