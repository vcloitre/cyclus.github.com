CEP 3 - |Cyclus| Release Procedure
********************************************************

:CEP: 3
:Title: |Cyclus| Release Procedure
:Last-Modified: 2015-05-01
:Author: Anthony Scopatz and Matthew Gidden
:Status: Accepted
:Type: Process
:Created: 2013-10-25

Abstract
========
The purpose of this document is to act as a guideline and checklist for how 
to release the |cyclus| core code base and the supported projects in the ecosystem.

The |Cyclus| Ecosystem
======================
The very first thing to do when preparing for an upcoming release is to elect 
a release manager.  This person has the special responsibility of making sure 
all of the following tasks are implemented.  Therefore, their judgment for the 
placement of issues and code stability must be adhered to.  

The |cyclus| ecosystem has a few projects which are all released together. 
(This may change in the future a development diverges and the core becomes more 
stable.)  The projects that are under the release manager's purview are:

* `Cyclus`_ 
* `Cycamore`_ 
* `Cycstub`_
* `Cymetric`_

The projects which are not yet under the release managers purview are:

* `Cyclist`_ 

Release Candidates (Tags & Branches)
====================================
At the beginning of a release, a special branch for *each* project should be
made off of ``develop`` named ``vX.X.X-release``. Note the *v* at the beginning. Each
project should have the initial version of of it's release branch *tagged* as
``X.X.X-rc1``, the first release candidate.

.. note:: 

    To distingush them, branch names have a ``v`` prefix (``vX.X.X-release``)
    while tag names lack this prefix (``X.X.X-rcX``).

Release candidates serve as an official proving ground for the release. Upon
creation, an announcement should be made to the developer's list, and users of
the project should be encouraged to test them out in order to bugs/other issues.

Any required changes must be pull requested from a topical branch into the
*release* branch.  After this has been accepted, the topical branch must be
merged with ``develop`` as well. The release branch is there so that development
can continue on the ``develop`` branch while the release candidates (rc) are out
and under review.  This is because otherwise any new developments would have to
wait until post-release to be merged into ``develop`` to prevent them from
accidentally getting released early.

Everything that is in the release branch must also be part of ``develop``.
Graphically,

.. figure:: cep-0003-1.svg
    :align: center

    **Figure 1:** Branch hierarchy under release.

.. note:: 

    Any commits merged into the release branch must *also* be merged into
    ``develop``. It is common practice for the release manager to request the
    reviewer pull requests to merge the subject topical branch into ``develop``
    as well. However, it is the ultimate release manager's responsibility to
    make sure ``develop`` is kept up to date with the ``release`` branch.

If changes are made to the release branch, a new candidate must be issued after
*2 - 5 days*. Every time a new release candidate comes out the ``vX.X.X-release``
must be tagged with the name ``X.X.X-rcX``. A developer's list annoucement must
accompany any new candidate.

The release branch must be quiet and untouched for *2 - 5 days prior* to the
full release. When the full and final release happens, the ``vX.X.X-release``
branch is merged into ``master`` and then deleted. All commits in the
``vX.X.X-release`` branch must have also been merged into the ``develop`` branch
as they were accepted.

Project Checklist
=================
.. note::

    Utility scripts for this process can be found in the `release`_ repository

When releasing a |cyclus| project, make sure to do the following items in order:

#. Review **ALL** issues and pull requests, reassigning or closing them as needed.

#. Ensure that all issues/PRs in this release's milestone have been closed.
   Moving them to the next release's milestone is a perfectly valid strategy for
   completing this milestone.

#. Initiate the release candidate process (see above)

#. Draft release notes

   - the ``make_release_notes.sh`` utility in ``release/utils`` will help
     provide a template

   .. code-block:: bash

      $ cd /path/to/release/utils
      $ export CYCLUS_DIR=/path/to/cyclus
      $ export CYCAMORE_DIR=/path/to/cycamore
      $ export CYMETRIC_DIR=/path/to/cymetric
      $ ./make_release_notes.sh W.W.W X.X.X # W.W.W is the previous version, X.X.X is *this* version

   - add the release notes to ``cyclus.github.com/source/previous/`` with
     appropriate updates to ``index.rst`` in that directory

#. Review the current state of documentation and make approriate updates.

#. Finish the release candidate process

   - make sure all commits in the ``release`` branch also are in ``develop``

#. Bump the version in ``cyclus/src/version.h``, commit the change

#. Perform maintainence tasks for this project

   - they are described in detail below, *but*
  
   - the ``maintenence.sh`` utility in ``release/utils`` will do this
     automatically for you

   .. code-block:: bash

      $ cd /path/to/release/utils
      $ export CYCLUS_DIR=/path/to/cyclus
      $ export CYCAMORE_DIR=/path/to/cycamore
      $ export CYCSTUB_DIR=/path/to/cycstub
      $ ./maintenence.sh -r -v X.X.X # X.X.X is *this* version

#. Upload the conda packages

   - the ``upload_conda.sh`` utility in ``release/utils`` will do this
     automatically for you

   .. code-block:: bash

      $ cd /path/to/release/utils
      $ export CYCLUS_DIR=/path/to/cyclus
      $ export CYCAMORE_DIR=/path/to/cycamore
      $ export CYMETRIC_DIR=/path/to/cymetric
      $ ./conda_upload.sh X.X.X # X.X.X is *this* version

#. Update the ``master`` branch of all projects

   - merge the ``release`` branch into ``master``
  
   - tag the master branch with the name 'X.X.X'

   - push the master branch and tag upstream

   - delete the release branch

#. Create a DOI. See :doc:`CEP4 <./cep4>` for details.

#. Update release information on the front page (``index.rst``) of the website.

Maintainence Tasks
==================
.. note::

    There is now the ``maintenence.sh`` utility in ``release/utils`` that
    will automate this for you. The section remains here for posterity.

Each project may have associate maintenance tasks which may need to be performed
at least as often as every micro release. 

|Cyclus|
--------
**Update PyNE:**  PyNE source code is included and shipped as part of |cyclus|. As pyne
evolves, we'll want to have our version evolve as well. Here are the steps to do so.
These assume that in your HOME dir there are both the pyne and |cyclus| repos.  Remember 
to check in the changes afterwards.

.. code-block:: bash

    $ cd ~/pyne
    $ ./amalgamate.py -s pyne.cc -i pyne.h
    $ cp pyne.* ~/cyclus/src
    
**Update Nuclear Data:** PyNE also provides a nuclear data library generator which we use for 
our source data.  Occassionally, this needs to be updated as updates to pyne itself come out.
The command for generating |cyclus| specific nuclear data is as follows:

.. code-block:: bash

   $ cd ~/pyne
   $ nuc_data_make -o cyclus_nuc_data.h5 \
    -m atomic_mass,scattering_lengths,decay,simple_xs,materials,eaf,wimsd_fpy,nds_fpy

Once the file is generated it must be put onto rackspace.

**Update Gtest:** We include a copy of the fused Gtest source code within our 
source tree located in the ``tests/GoogleTest`` directory.  To keep up with 
Gtest's natural evolution cycle, please download the latest release of Google Tests 
and follow `the fused source directions here`_.  If we go too long without doing this, 
it could be very painful to update.

**Verify & Update API Stability:** Since |Cyclus| v1.0 we promise API stability. 
Luckily, we have a tool for keeping track of this mostly automatically.  
Every release please run the following command to verify that the release 
branch is stable:

.. code-block:: bash

    $ cd cyclus/release
    $ ./smbchk.py --update -t HEAD --no-save --check

If |cyclus| only has API additions, it is considered stable and the command will 
tell you so. If |cyclus| also has API deletions, then |cyclus| is considered 
unstable and a diff of the symbols will be prinited. 
**You cannot release |cyclus| if it is unstable!** Please post the diff to 
either the mailing list or the issue tracker and work to resolve the removed
symbols until it this command declares that |cyclus| is stable. It is 
probably best to do this prior to any release candidates if possible.

Once stable and there are no more code changes to be made, add the symbols
in this release to the database with the following command:

.. code-block:: bash

    $ cd cyclus/release
    $ ./smbchk.py --update -t X.X.X

where ``X.X.X`` is the version tag. This should alter the ``symbols.json`` 
file.  Commit this and add it to the repo.  

Cycamore
--------
No maintenence required.

Cycstub
--------
Every release the relevant files from |cyclus| should be copied over to |cyclus|.
Use the following BASH commands to do so:

.. code-block:: bash

   $ cp ~/cyclus/tests/input/stub_example.xml ~/cycstub/input/example.xml && \
     cp ~/cyclus/stubs/stub_* ~/cycstub/src/

Cymetric
--------
No maintenance required.

Document History
================
This document is released under the CC-BY 3.0 license.

.. _Cyclus: https://github.com/cyclus/cyclus
.. _Cycamore: https://github.com/cyclus/cycamore
.. _Cycstub: https://github.com/cyclus/cycstub
.. _Cymetric: https://github.com/cyclus/cymetric
.. _Cyclist: https://github.com/cyclus/cyclist2
.. _release: https://github.com/cyclus/release
.. _the fused source directions here: https://code.google.com/p/googletest/wiki/V1_6_AdvancedGuide#Fusing_Google_Test_Source_Files
