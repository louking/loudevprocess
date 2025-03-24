Development Environment
++++++++++++++++++++++++++++++++

This document assumes a windows based development environment.

Required Software
-----------------------------

* visual studio code (https://code.visualstudio.com/)
* github desktop (https://desktop.github.com/) - you'll probably need a github account first
* python (https://www.python.org/) - as of 2/28/25, 3.12 is the version used for development
* docker desktop (https://www.docker.com/get-started/) 

Required account access
---------------------------
* github (https://github.com/)

Development System Configuration
-------------------------------------

* create ``config`` directory

  * get examples from Lou
  * create ``<app>.cfg``
  * create ``users.cfg``
  * create ``db`` directory (this will get "secrets" files which contain the passwords for the database(s))

    * https://www.lastpass.com/features/password-generator is a good way to generate passwords, 
      best not top use symbols as sometimes they cause issues 

  * create ``db_init`` directory (this will get sql import files -- note the sql file gets deleted after import)

* create ``.env`` file (get example from Lou)

  * update ``*_HOST`` variables to match your development environment

* create and populate python virtual env (https://docs.python.org/3/library/venv.html)

  use the following or let vscode do it for you

  .. code-block:: shell

    python3 -m venv .venv
    .venv\scripts\activate # or on linux source venv/bin/activate
    pip install -r requirements.txt

* create and populate databases

  * ``.env`` file variables are used to name and create the database
  * get sql import file(s) from Lou -- these go into the de_init 

Shell file permissions
--------------------------
If a shell file is created in a Windows development environment, it won't have execute permission when pushed to 
a Linux target. See http://blog.lesc.se/2011/11/how-to-change-file-premissions-in-git.html

After creating and committing the file, change its permissions in git

  .. code-block:: shell

    git update-index --chmod=+x .\app\src\dbupgrade_and_run.sh

then commit as normal

Docker files
--------------
Example docker files can be found at https://github.com/louking/webmodules, with the latest docker-skeleton-vx.x tag

vscode launch.json
--------------------
For debugging, you'll need the following in vscode's launch.json. Note that the existing repos all have launch.json and task.json files

  .. code-block:: shell

    // https://code.visualstudio.com/docs/containers/docker-compose#_python
    {
        "name": "Python: Remote Attach",
        "type": "python",
        "request": "attach",
        "port": 5678,
        "host": "localhost",
        "pathMappings": [
            {
                "localRoot": "${workspaceFolder}/app/src",
                "remoteRoot": "/app"
            }
        ],
        "justMyCode": false
    },

Development vs Production via docker compose
-------------------------------------------------

Build and start app in development, no debugging
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: shell

    docker compose -f docker-compose.yml -f docker-compose.dev.yml up --build -d

or ctrl-p task up (or task dev)

Build and start app in development, with debugging
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Run

  .. code-block:: shell

    docker compose -f docker-compose.yml -f docker-compose.dev.yml -f docker-compose.debug.yml up --build -d

then start debugger with vscode 

Build and start app in Production
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: shell

    docker compose -f docker-compose.yml -f docker-compose.prod.yml up --build -d



Kanban Board
---------------
Contact Lou to get read/write access to the repo's kanban board

Development Workflow
-----------------------

See https://docs.github.com/en/get-started/quickstart/contributing-to-projects

Synopsys:

* fork repository on GitHub
* clone fork on development workstation
* create a branch for a given change
* test change in development environment
* commit change to branch -- title should be annoted with "(issue #)"
* push change to forked repository
* generate a pull request
* mark issue as fixed
