===========
cloudinit.d
===========

cloudinit.d is a tool designed for launching, controlling, and monitoring
complex environments in the cloud.  Its most important feature is repeatable,
one-click, deployment of sets of VMs configured with *launch plans*.  These
sets of VMs can be deployed over multiple clouds (Eucalyptus, Nimbus,
OpenStack, and Amazon EC2 are currently supported) as well as include
non-virtualized resources.  Like the Unix init.d process, cloudinit.d can
manage dependencies between deployed VMs.  It also provides mechanisms for
testing, monitoring, and repairing a launch.  For more information about
cloudinit.d see the pages below and our `Teragrid 2011 paper
<http://www.nimbusproject.org/files/cloudinitd_tg11_submit3c.pdf>`_.
For repeatable experiment management with cloudinit.d read the `report on
support for experimental computer science
<http://www.nimbusproject.org/downloads/Supporting_Experimental_Computer_Science_final_draft.pdf>`_.

.. toctree::
   :maxdepth: 1

   intro
   quickstart
   wordpress
   cloudfoundry
   service
   install
   deps
   monitor
   launchplans

.. image:: images/cloudinitd_dia.png
   :width: 600
   :height: 400
