python Package Release Procedures
++++++++++++++++++++++++++++++++++++

release to github
-----------------

-  before updating version

   -  update readme file

-  check if update to setup.py needed (for new scripts - 3 places)

-  pip freeze > requirements.txt

-  commit requirements.txt before the following

   -  git add requirements.txt

   -  git commit -m 'package requirements update'

-  update version.py

-  <package>\setup.py install

-  commit change "version x.y.z"

   -  git add <package>/version.py

   -  git commit -m 'version x.y.z'

-  if branch

   -  merge branch to master

      -  git checkout master

      -  git merge <branchname>

      -  see
            https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging

-  sync master to github (git push)

-  in branch shell

   -  git tag x.y.z -m 'version x.y.z'

   -  git push --tags

-  create documentation

   -  git log --oneline 2.0.10..2.0.11 [lastrelease..thisrelease]

   -  on github

      -  click <> code

      -  click releases

      -  click Draft a new release button

      -  reformat output from git log above, into release notes

-  If branch, delete remote and local versions **[best if you wait to do
      this until after deploy, if a branch was deployed earlier]**

   -  git push origin --delete <branchname> # delete remote

   -  git branch -d <branchname> # delete local

   -  if see the following, try git checkout master at target

      -  [scoretility@sandbox.scoretility.com] out: Your configuration
            specifies to merge with the ref '<branchname>'

      -  [scoretility@sandbox.scoretility.com] out: from the remote, but
            no such ref was fetched.

   -  

release to PyPi
---------------

test release with editable install
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To test with another package which may be changing

-  see https://pip.pypa.io/en/stable/reference/pip_install/ “Editable
      Installs”

-  pip uninstall <package>

-  pip install -e "C:\Users\lking\Documents\Lou's
      Software\projects\loutilities\loutilities"

release
~~~~~~~

-  see https://packaging.python.org/tutorials/packaging-projects/

-  for test

   -  set version to x.y.z.\ **devn**

-  python setup.py install sdist bdist_wheel

-  twine upload dist/<package>-<version>.\*