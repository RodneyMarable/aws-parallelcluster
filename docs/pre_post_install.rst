.. _pre_post_install:

Custom Bootstrap Actions
========================

AWS ParallelCluster can execute arbitrary code either before(pre) or after(post) the main bootstrap action during
cluster creation. This code is typically stored in S3 and accessed via HTTP(S) during cluster creation. The code will
be executed as root and can be in any script language supported by the cluster OS, typically `bash` or `python`.

pre-install actions are called before any cluster deployment bootstrap such as configuring NAT, EBS and the scheduler.
Typical pre-install actions may include modifying storage, adding extra users or packages.

post-install actions are called after cluster bootstrap is complete, as the last action before an instance is
considered complete. Typical post-install actions may include changing scheduler settings, modifying storage or
packages.

Arguments can be passed to scripts by specifying them in the config. These will be passed double-quoted to the
pre/post-install actions.

If a pre/post-install actions fails, then the instance bootstrap will be considered failed and it will not continue.
Success is signalled with an exit code of 0, any other exit code will be considered a fail.

Configuration
-------------

The following config settings are used to define pre/post-install actions and arguments. All options are optional and
are not required for basic cluster install.

::

    # URL to a preinstall script. This is executed before any of the boot_as_* scripts are run
    # (defaults to NONE)
    pre_install = NONE
    # Arguments to be passed to preinstall script
    # (defaults to NONE)
    pre_install_args = NONE
    # URL to a postinstall script. This is executed after any of the boot_as_* scripts are run
    # (defaults to NONE)
    post_install = NONE
    # Arguments to be passed to postinstall script
    # (defaults to NONE)
    post_install_args = NONE

Arguments
---------
The first two arguments ``$0`` and ``$1`` are reserved for the script name and url.

::

    $0 => the script name
    $1 => s3 url
    $n => args set by pre/post_install_args

Example
-------

The following are some steps to create a simple post install script that installs the R packages in a cluster.

1. Create a script. For the R example, see below

::

    #!/bin/bash

    yum -y install --enablerepo=epel R

2. Upload the script with the correct permissions to S3

``aws s3 cp --acl public-read /path/to/myscript.sh s3://<bucket-name>/myscript.sh``

3. Update AWS ParallelCluster config to include the new post install action.

::

    [cluster default]
    ...
    post_install = https://<bucket-name>.s3.amazonaws.com/myscript.sh

If the bucket does not have public-read permission use ``s3`` as URL scheme.

::

    [cluster default]
    ...
    post_install = s3://<bucket-name>/myscript.sh


4. Launch a cluster

``pcluster create mycluster``
