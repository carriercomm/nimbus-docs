==========
Quickstart
==========

Download
========

cloudinit.d is a python setup package registered with
`PyPI <http://pypi.python.org/pypi>`_ which means you can ``easy_install``
it.  From a python ``virtualenv`` or an account with access to the system
libraries and python in its path run::

    $ easy_install cloudinitd

This will download cloudinit.d and all of its dependencies and install them.

.. todo::
    We need the real download link 
    `here <http://www.nimbusproject.org/downloads/cloudinitd-1.2.tar.gz>`_. Hopefully in some automated way?

If you prefer to fetch the tarball and have a more manual install you can
get the latest release `here <http://www.nimbusproject.org/downloads/cloudinitd-1.2.tar.gz>`_.  To install it::

    $ tar -zxf cloudinitd-latest.tar.gz
    $ cd cloudinitd-<version>
    $ python setup.py install

You can now test it out! ::

    $ cloudinitd --help
    Usage: [options] <command> [<top level launch plan> | <run name>]
    Boot and manage a launch plan
    Run with the command 'commands' to see a list of all possible commands


    Options:
      --version             show program's version number and exit
      -h, --help            show this help message and exit
      -v, --verbose         Print more output
      -x, --validate        Check that boot plan is valid before launching it.
      -y, --dryrun          Perform dry run on the boot plan.  The IaaS service is
                            never contacted but all other actions are performed.
                            This option offers an addition level of plan
                            validation of -x.
      -q, --quiet           Print no output
      -n NAME, --name=NAME  Set the run name, only relevant for boot and reload
                            (by default the system picks)
      -d DATABASE, --database=DATABASE
                            Path to the db directory
      -f LOGDIR, --logdir=LOGDIR
                            Path to the base log directory.
      -l LOGLEVEL, --loglevel=LOGLEVEL
                            Controls the level of detail in the log file : {debug
                            | info | warn | error}
      -s, --logstack        Log stack trace information (extreme debug level)
      -c, --noclean         Do not delete the database, only relevant for the
                            terminate command
      -k, --kill            This option only applies to the iceage command.  When
                            on it will terminate all VMs started with IaaS
                            associated with this run to date.  This should be
                            considered an extreme measure to prevent IaaS resource
                            leaks.
      -o OUTPUT, --output=OUTPUT
                            Create an json document which describes the
                            application and write it to the associated file.
                            Relevant for boot and status
      -g GLOBALVAR, --globalvar=GLOBALVAR
                            Add a variable to global variable space
      -G GLOBALVARFILE, --globalvarfile=GLOBALVARFILE
                            Add a file to global variable space


Create a launch plan
====================

A launch plan is a set of configuration files that tell cloudinit.d what
VM images to boot and in what order.  It is where the specific cloud
to use is set along with the credentials needed to access that cloud.
All aspects of contextualization and testing are also set in the
launch plan.

Here we will walk through a very basic launch plan.  To minimize potential
confusion we will only show a bare minimum set of cloudinit.d features.

Simple EC2 example
------------------

In this example we show a minimal launch plan needed to start a
VM on Amazon's EC2.  In order to use it you will need an account with
`AWS <https://aws.amazon.com>`_.

.. note::
    For this example to work you need your default security group to have
    port 22 and 80.

The launch plan files used in this example can be downloaded:

* :download:`helloec2.conf <helloec2.conf>`
* :download:`helloec2_level1.conf <helloec2_level1.conf>`

Top level launch plan
---------------------

In this very simple example the top level configuration file simply enumerates
the run levels.  Because we are only launching a single virtual machine
there is only one run level.  The entire configuration file follows::

    [runlevels]
    level1: helloec2_level1.conf

Run level configuration
-----------------------

In this example all of the interesting information will be held in the
run level configuration file.  We will need to have the following
information in order to run this example.

* EC2 access key
* EC2 secret key
* EC2 ssh key name
* path to the matching ssh key

These are all very standard things that EC2 users have.  If you are not
familiar with these items please check out the 
`EC2 tutorial <http://docs.amazonwebservices.com/AWSEC2/latest/GettingStartedGuide/>`_.

Once you have obtained the above information please set the following
environment variables in the following way::
    
    $ export CLOUDINITD_IAAS_ACCESS_KEY=<your EC2 access key>
    $ export CLOUDINITD_IAAS_SECRET_KEY=<your EC2 secret key>
    $ export CLOUDINITD_IAAS_SSHKEY=<the path to your ssh key>
    $ export CLOUDINITD_IAAS_SSHKEYNAME=<the name of your ssh key in EC2>

Now that we have all of the security information in place we will look
at the run level configuration file:

.. literalinclude:: helloec2_level1.conf

The name of this configuration section is ``svc-sampleservice``.
What this tells cloudinit.d is that it is a service (svc-) named
sampleservice.  The name must start with ``svc-``.

The first thing to note from this file are the values corresponding to the
environment variables we just set.  The directive ``env.`` tells cloudinit.d
to get the value for this field from the environment variable of the
following name.  The values for these 4 entries are described above.

The remaining entries have the following meanings:

* ``iaas`` This is a string describing the cloud with which we will be
  interacting.  In this case it is us-east-1 on EC2.
* ``image`` This is a handle to an image.  The format of this handle
  will vary with different clouds.  In our case we are using EC2 so the
  format is ``ami-XXXXXXX``.  The specific AMI for use with this example is
  ``ami-6fa27506``.  It is a publicly available Ubuntu image. Note that this
  AMI is managed by the Ubuntu project and could become unavailable in the
  future.  In such case, you can simply pick a new AMI from the `AMI locator
  page <http://cloud.ubuntu.com/ami/>`_.
* ``ssh_username`` This is the username that the VM image has associated with 
  the ``localsshkeypath`` defined above.
* ``allocation`` This string tells the cloud details of the resources
  we want allocated with the image.  In our case we pick t1.micro because
  it is the cheapest.
* ``bootpgm`` This value is described in the next section.


Boot program
------------

This value points to a program that will be run on your virtual machine once to
contextualize it.  Once you boot your launch program cloudinit.d will wait
until its sshd service is running.  Once it is this program is uploaded and run
as the user defined by ``ssh_username``, in our case ``ubuntu``.  Below is the
example bootpgm:

.. literalinclude:: ec2bootpgm.py

This is a very simple program which simply installs the Apache web server and
creates a web page.  You can also download it here: :download:`ec2bootpgm.py
<ec2bootpgm.py>`.

Boot
====

Now that we have and understand a launch plan, let's put it into action with cloudinit.d.

.. note:: You will be charged by AWS for running this command.  At the time
    of this writing the charges are less than 3 cents per hour.

::

    $ cloudinitd -v -v -v boot helloec2.conf
    Starting up run 54c03415
        Started IaaS work for sampleservice
    Starting the launch plan.
    Begin boot level 1...
        Started sampleservice
        SUCCESS service sampleservice boot
            hostname: ec2-50-17-4-154.compute-1.amazonaws.com
            instance: i-2ae3a245
    SUCCESS level 1

The above session shows a successful boot of a launch plan.  Notice the use of
the ``-v`` option.  This simply increases the amount of output that we will
see.

Your VM was successfully started!  The output shows the hostname assigned to
the boot as **ec2-50-17-4-154.compute-1.amazonaws.com**.  If we point a
web browser at ``http://ec2-50-17-4-154.compute-1.amazonaws.com/hello.html``
we should see the message we configured with our ec2bootpgm.py program. Note
that your URL will be different.

Terminate
=========

In order to avoid further service charges from AWS we should clean up this
launch.  This is done by taking the run name from the output of the boot,
in our case ``54c03415`` and giving it as an option to the terminate directive::

    $ cloudinitd -v -v -v terminate 54c03415
    Terminating 54c03415
    Begin terminate level 1...
        Started sampleservice
        SUCCESS service sampleservice terminate
            hostname: None
            instance: None
    SUCCESS level 1
    deleting the db file /home/bresnaha/.cloudinitd/cloudinitd-54c03415.db


Example launch plans
====================

.. toctree::
   :maxdepth: 1

   wordpress
