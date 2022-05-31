Development Environment
++++++++++++++++++++++++++++++++

This document assumes a windows based development environment.

Required Software
-----------------------------

* mysql or mariadb

  * Suggest installing MAMP (https://www.mamp.info/). This also starts apache which isn't used, but provides phpMyAdmin for high level database Management

* visual studio code (https://code.visualstudio.com/)
* github desktop (https://desktop.github.com/) - you'll probably need a github account first
* python (https://www.python.org/)

Required account access
---------------------------
* github (https://github.com/)

Development System Configuration
-------------------------------------

* create config directory

  * get examples from Lou
  * create <app>.cfg
  * create users.cfg 

* create and populate python virtual env (https://docs.python.org/3/library/venv.html)

  .. code-block:: shell

    python3 -m venv venv
    pip install -r requirements.txt

* create and populate databases (:ref:`mysql-database-management`)

  * get sql import files from Lou