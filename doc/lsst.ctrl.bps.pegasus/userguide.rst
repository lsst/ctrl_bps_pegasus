.. _peg-plugin-overview:

Overview
--------

LSST Batch Processing Service (BPS) allows large-scale workflows to execute in
well-managed fashion, potentially in multiple environments.  The service is
provided by the `ctrl_bps`_ package.  ``ctrl_bps_pegauss`` is a plugin allowing
`ctrl_bps` to execute workflows on computational resources managed by `Pegasus
WMS`_.

.. _peg-plugin-preqs:

Prerequisites
-------------

#. `ctrl_bps`_, the package providing LSST Batch Processing Service.
#. `HTCondor`_ pool.
#. `Pegasus WMS`_ and its Python `bindings`__

.. __: https://htcondor.readthedocs.io/en/latest/apis/python-bindings/index.html

.. _peg-plugin-installing:

Installing the plugin
---------------------

Install the plugin similarly to any other LSST package:

.. code-block:: bash

   git clone https://github.com/lsst/ctrl_bps_pegasus
   cd ctrl_bps_htcondor
   setup -k -r .
   scons

.. _peg-plugin-wmsclass:

Specifying the plugin
---------------------

The class providing `Pegasus WMS`_ support for `ctrl_bps`_ is ::

    lsst.ctrl.bps.htcondor.PegasusService

Inform `ctrl_bps`_ about its location using one of the methods described in its
`documentation`__.

.. __: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/index.html

.. _peg-plugin-defining-submission:

Defining a submission
---------------------

BPS configuration files are YAML files with some reserved keywords and some
special features. See `BPS configuration file`__ for details.

The plugin supports all settings described in `ctrl_bps documentation`__
*except* **preemptible**.

.. Describe any plugin specific aspects of defining a submission below if any.

Under the hood `Pegasus WMS` uses `HTCondor`_ to manage execution of the workflow.
`HTCondor`_ is able to to send jobs to run on a remote compute site, even when
that compute site is running a non-HTCondor system, by sending "pilot jobs", or
**gliedins**, to remote batch systems.

Nodes for HTCondor's glideins can be allocated with help of `ctrl_execute`_.
Once you allocated the nodes, you can specify the site where there are
available in your BPS configuration file. For example:

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

.. __: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#bps-configuration-file
.. __: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#supported-settings

.. .. _peg-plugin-authenticating:

.. Authenticating
.. --------------

.. Describe any plugin specific aspects of an authentication below if any.

.. _peg-plugin-submit:

Submitting a run
----------------

See `bps submit`_.

.. Describe any plugin specific aspects of a submission below if any.

.. _peg-plugin-report:

Checking status
---------------

See `bps report`_.

.. Describe any plugin specific aspects of checking a submission status below
   if any.

.. _peg-plugin-cancel:

Canceling submitted jobs
------------------------

See `bps cancel`_.

.. Describe any plugin specific aspects of canceling submitted jobs below
   if any.

.. _peg-plugin-restart:

Restarting a failed run
-----------------------

`bps restart`_ is *not* supported, use the WMS commands/tools directly.

.. Describe any plugin specific aspects of restarting failed jobs below
   if any.

.. _peg-plugin-troubleshooting:

Troubleshooting
---------------

Where is stdout/stderr from pipeline tasks?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pegasus does its own directory structure and wrapping of ``pipetask`` output.

You can dig around in the submit run directory here too, but try
`pegasus-analyzer`_ command first.

The Pegasus ``runId`` is the submit subdirectory where all underlying files
lives.  If you've forgotten the Pegasus ``runId`` needed to use in the Pegasus
commands try one of the following:

#. It's the submit directory in which the ``braindump.txt`` file lives.  If you
   know the submit root directory, use find to give you a list of directories
   to try.  (Note that many of these directories could be for old runs that are
   no longer running.)

#. Use `HTCondor`_ commands to find submit directories for running jobs

.. code-block:: bash

   condor_q -constraint 'pegasus_wf_xformation == "pegasus::dagman"' -l | grep Iwd


.. _HTCondor: https://htcondor.readthedocs.io/en/latest/
.. _Pegasus WMS: https://pegasus.isi.edu/documentation/index.html
.. _bps cancel: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#canceling-submitted-jobs
.. _bps report: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#checking-status
.. _bps restart: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#restarting-a-failed-run
.. _bps submit: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#submitting-a-run
.. _ctrl_bps: https://github.com/lsst/ctrl_bps
.. _ctrl_execute: https://github.com/lsst/ctrl_execute
.. _pegasus-analyzer: https://pegasus.isi.edu/documentation/manpages/pegasus-analyzer.html
