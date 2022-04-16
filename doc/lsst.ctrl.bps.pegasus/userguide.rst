.. _peg-plugin-preqs:

Prerequisites
-------------

#. `ctrl_bps`_, the package providing LSST Batch Processing Service.

#. `HTCondor`_ cluster. NCSA hosts a few of such clusters, see `this`__ page for
   details.

#. `Pegasus`_ WMS and its Python `bindings`__

.. __: https://developer.lsst.io/services/batch.html
.. __: https://htcondor.readthedocs.io/en/latest/apis/python-bindings/index.html

.. _peg-plugin-installing:

Installing the plugin
---------------------

Install the Pegsus plugin similarly to any other LSST package:

.. code-block:: bash

   git clone https://github.com/lsst/ctrl_bps_pegasus
   cd ctrl_bps_htcondor
   setup -k -r .
   scons

.. warning::

   Under normal circumstances, we **strongly** recommend to use `ctrl_bps`
   commands for performing the operations described below first (run ``bps
   --help`` to see currently supported commads).  However, using Pegasus WMS
   commands directly may be useful to bypass `ctrl_bps`_ completely during
   debugging/troubleshooting.

.. _peg-plugin-status:

Checking status
---------------

To check the status of the submitted run use `pegasus_status`__ which takes
Pegasus submit directory.  For example:

.. code-block:: bash

   pegasus-status /scratch/submit/u/jdoe/pipelines_check/20220414T184635Z/peg/jdoe/pegasus/u_jdoe_pipelines_check_20220414T184635Z/run0001

.. __: https://pegasus.isi.edu/documentation/manpages/pegasus-status.html

.. _peg-plugin-cancelling:

Canceling submitted jobs
------------------------

To remove the jobs associated with a given submission from the job queue, use
`pegasus_remove`__ which takes the Pegasus submit directory.  For example:

.. code-block:: bash

   pegasus_remove /scratch/submit/u/jdoe/pipelines_check/20220414T184635Z/peg/jdoe/pegasus/u_jdoe_pipelines_check_20220414T184635Z/run0001

.. __: https://pegasus.isi.edu/documentation/manpages/pegasus-remove.html

.. _peg-plugin-settings:

Supported setttings
-------------------

Pegasus plugin supports all settings described in ``ctrl_bps`` `documentation`__ *except* **preemptible**.

.. __: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#supported-settings

.. _peg-plugin-troubleshooting:

Troubleshooting
---------------

Where is stdout/stderr from pipeline tasks?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pegasus does its own directory structure and wrapping of ``pipetask`` output.

You can dig around in the submit run directory here too, but try
`pegasus-analyzer`__ command first.

The Pegasus ``runId`` is the submit subdirectory where all underlying files
lives.  If you've forgotten the Pegasus ``runId`` needed to use in the Pegasus
commands try one of the following:

#. It’s the submit directory in which the ``braindump.txt`` file lives.  If you
   know the submit root directory, use find to give you a list of directories
   to try.  (Note that many of these directories could be for old runs that are
   no longer running.)

#. Use `HTCondor`_ commands to find submit directories for running jobs

.. code-block:: bash

   condor_q -constraint 'pegasus_wf_xformation == "pegasus::dagman"' -l | grep Iwd

How to match jobs to glideins?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Specify compute site as follow:

.. code-block:: YAML

   site:
     acsws02:
       arch: x86_64
       os: LINUX
       directory:
         shared-scratch:
           path: /work/shared-scratch/${USER}
           file-server:
             operation: all
             url: file:///work/shared-scratch/${USER}
       profile:
         pegasus:
           style: condor
           auxillary.local: true
         condor:
           universe: vanilla
           getenv: true
           requirements: '(ALLOCATED_NODE_SET == &quot;${NODESET}&quot;)'
           +JOB_NODE_SET: '&quot;${NODESET}&quot;'
         dagman:
           retry: 0
         env:
           PEGASUS_HOME: /usr/local/pegasus/current

.. __: https://pegasus.isi.edu/documentation/manpages/pegasus-analyzer.html


.. _HTCondor: https://htcondor.readthedocs.io/en/latest/
.. _Pegasus: https://pegasus.isi.edu/documentation/index.html
.. _ctrl_bps: https://github.com/lsst/ctrl_bps
