python Package Release Procedures
++++++++++++++++++++++++++++++++++++

release to github
-----------------

-  before updating version

   -  update readme file

.. Padding. See https://github.com/sphinx-doc/sphinx/issues/2258

-  check if update to setup.py needed (for new scripts - 3 places)
-  freeze requirements

.. code-block:: shell

    pip freeze > requirements.txt

or using powershell

.. code-block:: shell

    pip freeze | out-file .\app\requirements.txt -encoding ascii
    pip freeze | out-file .\requirements.txt -encoding ascii

-  commit requirements.txt before the following

.. code-block:: shell

   git add requirements.txt
   git commit -m 'package requirements update'

-  update version.py

.. code-block:: shell

    <package>\setup.py install

-  commit change "version x.y.z"

.. code-block:: shell

   git add <package>/version.py
   git commit -m 'version x.y.z'

-  if branch

   -  merge branch to master. see https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging

.. code-block:: shell

      git checkout master
      git merge <branchname>

-  sync master to github (git push)
-  in branch shell

.. code-block:: shell

   git tag x.y.z -m 'version x.y.z'
   git push --tags

-  create documentation

.. code-block:: shell

   git log --oneline 2.0.10..2.0.11 [lastrelease..thisrelease]

-  create documentation (con'd)

   -  on github

      -  click <> code
      -  click releases
      -  click Draft a new release button
      -  reformat output from git log above, into release notes

-  If branch, delete remote and local versions **[best if you wait to do this until after deploy, if a branch was deployed earlier]**

.. code-block:: shell

   -  git push origin --delete <branchname> # delete remote
   -  git branch -d <branchname> # delete local

-  if see the following, try git checkout master at target

      -  [scoretility@sandbox.scoretility.com] out: Your configuration specifies to merge with the ref '<branchname>'
      -  [scoretility@sandbox.scoretility.com] out: from the remote, but no such ref was fetched.


sync your fork
----------------

If you have a :term:`fork` of the :term:`upstream` :term:`repo <repository>`, you'll need to sync that :term:`fork`
periodically.

-   see https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/syncing-a-fork
-   using PyCharm

    -   checkout master [if not already checked out -- see lower right of PyCharm project view]
    -   VCS > Git > Rebase my GitHub fork > upstream [you'll need to log in to github the first time you do this]

        -   if there are merge conflicts, decide on whether to accept yours, accept theirs, or merge

    -   VCS > Git > Merge Changes... > remotes/upstream/master [merges :term:`upstream`/master into local master branch]
    -   VCS > Git > Push... [pushes local master branch to fork (:term:`origin`)]






release to PyPi
---------------

test release with editable install
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To test with another package which may be changing

-  see https://pip.pypa.io/en/stable/reference/pip_install/ "Editable Installs"

.. code-block:: shell

    pip uninstall <package>
    pip install -e "C:\Users\lking\Documents\Lou's Software\projects\loutilities\loutilities"

create .pypirc
~~~~~~~~~~~~~~~~~~~
in C:\Users\<username> create .pypirc file

.. code-block:: cfg

    [distutils]
    index-servers=
        pypi

    [pypi]
        username = __token__
        password = <api token from pypi.org/manage/account>

release
~~~~~~~

-  see https://packaging.python.org/tutorials/packaging-projects/

-  for test

   -  set version to x.y.z.\ **devn**

.. code-block:: shell

    python -m build
    twine upload dist/<package>-<version>*.*

alternately create a task in tasks.json

.. code-block:: javascript

	"tasks": [
        {
            "label": "push to pypi",
            "type": "shell",
            "command":"venv/scripts/activate; python -m build; twine upload dist/<package>-${input:packageVersion}.tar.* dist/loutilities-${input:packageVersion}-*.*",
            "problemMatcher": []
        }
    ],

    "inputs": [
        {
            "id": "packageVersion",
            "type": "promptString",
            "description": "Enter version string",
        }
    ]

and use ctrl-p task push to pypi to push the task

- then force install

.. code-block:: shell

    pip install --force-reinstall <package>

Initial deploy to server
--------------------------
Log into server sudo account

Create server directory structure and virtual environment

.. code-block:: shell

    ### upload webapp files to target host
    sudo mkdir -p /var/www/www.<vhost>/<repo-name>
    cd /var/www/www.<vhost>/<repo-name>
    sudo git clone https://github.com/louking/<repo-name>
    cd /var/www/www.<vhost>
    sudo chown -R <vhostuser>:<vhostuser> <repo-name>
    sudo mkdir /var/www/www.<vhost>/applogs
    sudo chown -R <vhostuser>:<vhostuser> /var/www/www.<vhost>/applogs
    sudo mkdir /var/www/www.<vhost>/<repo-name>/<repo-name>/config
    sudo chown -R <vhostuser>:<vhostuser> /var/www/www.<vhost>/<repo-name>/<repo-name>/config

    ### Create python virtual environment
    cd /var/www/www.<vhost>
    sudo mkdir venv
    sudo chown -R <vhostuser>:<vhostuser> venv
    sudo su <vhostuser>
    (<vhostuser>> python3 -m venv venv
    (<vhostuser>) exit
    # see https://bugs.python.org/issue21496,
    # since venv wasn't created from virtualenv, activate_this.py is missing
    # needs to be present for wsgi application to work
    sudo cp /home/lking/activate-this/activate_this.py venv/bin
    sudo chown -R <vhostuser>:apache venv/bin/activate_this.py
    sudo su <vhostuser>
    (<vhostuser>) source venv/bin/activate
    (<vhostuser>) pip install --upgrade pip
    (<vhostuser>) cd /var/www/www.<vhost>/<repo-name>/<repo-name>
    (<vhostuser>) pip install -r requirements.txt

Create databases

- see https://loudevprocess.readthedocs.io/en/latest/mysql-database-management.html

Create javascript libraries

.. code-block:: shell

    sudo mkdir /var/www/<vhost>/libs
    sudo chown <vhostuser>:<vhostuser> /var/www/<vhost>/libs

- copy from development static/js to /var/www/<vhost>/libs/js

.. _python-ongoing-development:

Ongoing Development
--------------------------

target hosts are

* www.<slug>.loutilities.com
* sandbox.<slug>.loutilities.com

where

    slug
        is like routes, contracts, scores, etc

.. note::
    scores targets are initially scoretility.com, sandbox.scoretility.com, beta.scoretility.com

.. warning::
    before releasing to production, test using ASSETS_DEBUG: False

for official releases use fab

.. note::
    may need to copy/adjust fabric.json from another project

.. code-block:: shell

    fab -H <target-host> deploy

or

.. code-block:: shell

    fab -H <target1>,<target2> deploy

then on target system

.. code-block:: shell

    sudo systemctl restart vhost-membertility-www.service #for example

if you need to check out a particular branch. Note <branch> can be a tag, e.g., to downgrade

.. code-block:: shell

    fab -H <target-host> deploy --branchname=<branch>

for testing use winscp to load patch files, only to sandbox and possibly beta

* after testing the patch be sure to git checkout the original file, then use fab for clean upgrade


PyCharm Licencing
------------------

see https://sales.jetbrains.com/hc/en-gb/articles/207739199-Distributing-Commercial-Licenses