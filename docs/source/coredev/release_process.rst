.. _release_process:

=======================
IPython release process
=======================

This document contains the process that is used to create an IPython release.

Conveniently, the ``release`` script in the ``tools`` directory of the ``IPython``
repository automates most of the release process. This document serves as a
handy reminder and checklist for the release manager. 

During the release process, you might need the extra following dependencies:

 - ``keyring`` to access your GitHub authentication tokens
 - ``graphviz`` to generate some graphs in the documentation
 
Make sure you have all the required dependencies to run the tests as well.
 

1. Set Environment variables
----------------------------

Set environment variables to document previous release tag, current
release milestone, current release version, and git tag.

These variables may be used later to copy/paste as answers to the script
questions instead of typing the appropriate command when the time comes. These
variables are not used by the scripts directly; therefore, there is no need to
``export`` them. The format for bash is as follows, but note that these values
are just an example valid only for the 5.0 release; you'll need to update them
for the release you are actually making::

    PREV_RELEASE=4.2.1
    MILESTONE=5.0
    VERSION=5.0.0
    BRANCH=master


2. Create GitHub stats and finish release note
----------------------------------------------

.. note::

    This step is optional if making a Beta or RC release.

.. note::

    Before generating the GitHub stats, verify that all closed issues and pull
    requests have `appropriate milestones
    <https://github.com/ipython/ipython/wiki/Dev%3A-GitHub-workflow#milestones>`_.
    `This search
    <https://github.com/ipython/ipython/issues?q=is%3Aclosed+no%3Amilestone+is%3Aissue>`_
    should return no results before creating the GitHub stats.

If a major release:

    - merge any pull request notes into what's new::

          python tools/update_whatsnew.py

    - update ``docs/source/whatsnew/development.rst``, to ensure it covers
      the major release features

    - move the contents of ``development.rst`` to ``versionX.rst`` where ``X`` is
      the numerical release version

    - generate summary of GitHub contributions, which can be done with::

          python tools/github_stats.py --milestone $MILESTONE > stats.rst

      which may need some manual cleanup of ``stats.rst``. Add the cleaned
      ``stats.rst`` results to ``docs/source/whatsnew/github-stats-X.rst``
      where ``X`` is the numerical release version (don't forget to add it to
      the git repo as well). If creating a major release, make a new
      ``github-stats-X.rst`` file; if creating a minor release, the content
      from ``stats.rst`` may simply be added to the top of an existing
      ``github-stats-X.rst`` file.  Finally, edit
      ``docs/source/whatsnew/index.rst`` to list the new ``github-stats-X``
      file you just created and remove temporarily the first entry called
      ``development`` (you'll need to add it back after release).

      Make sure that the stats file has a header or it won't be rendered in
      the final documentation. 

To find duplicates and update `.mailmap`, use::

    git log --format="%aN <%aE>" $PREV_RELEASE... | sort -u -f

3. Make sure the repository is clean
------------------------------------

of any file that could be problematic.
   Remove all non-tracked files with:

   .. code::

       git clean -xfdi

   This will ask for confirmation before removing all untracked files. Make
   sure the ``dist/`` folder is clean to avoid any stale builds from
   previous build attempts.


4. Update the release version number
------------------------------------

Edit ``IPython/core/release.py`` to have the current version.

in particular, update version number and ``_version_extra`` content in
``IPython/core/release.py``.

Step 5 will validate your changes automatically, but you might still want to
make sure the version number matches pep440.

In particular, ``rc`` and ``beta`` are not separated by ``.`` or the ``sdist``
and ``bdist`` will appear as different releases. For example, a valid version
number for a release candidate (rc) release is: ``1.3rc1``. Notice that there
is no separator between the '3' and the 'r'. Check the environment variable
``$VERSION`` as well.

You will likely just have to modify/comment/uncomment one of the lines setting
``_version_extra``


5. Run the `tools/build_release` script
---------------------------------------

Running ``tools/build_release`` does all the file checking and building that
the real release script will do. This makes test installations, checks that
the build procedure runs OK, and tests other steps in the release process.

The ``build_release`` script will in particular verify that the version number
match PEP 440, in order to avoid surprise at the time of build upload.

We encourage creating a test build of the docs as well.

6. Create and push the new tag
------------------------------

Commit the changes to release.py::

    git commit -am "release $VERSION"
    git push origin $BRANCH

Create and push the tag::

    git tag -am "release $VERSION" "$VERSION"
    git push origin --tags

Update release.py back to ``x.y-dev`` or ``x.y-maint``, and re-add the
``development`` entry in ``docs/source/whatsnew/index.rst`` and push::

    git commit -am "back to development"
    git push origin $BRANCH

7. Get a fresh clone
--------------------

Get a fresh clone of the tag for building the release::

    cd /tmp
    git clone --depth 1 https://github.com/ipython/ipython.git -b "$VERSION"
    cd ipython

.. note::

    You can aslo cleanup the current working repository with ``git clean -xfdi``

8. Run the release script
-------------------------

Run the ``release`` script, this step requires having a current wheel, Python
>=3.4 and Python 2.7.::

    ./tools/release

This makes the tarballs, zipfiles, and wheels, and put them under the ``dist/``
folder. Be sure to test the ``wheels``  and the ``sdist`` locally before
uploading them to PyPI. We do not use an universal wheel as each wheel
installs an ``ipython2`` or ``ipython3`` script, depending on the version of
Python it is built for. Using an universal wheel would prevent this.

Use the following to actually upload the result of the build::

    ./tools/release upload

It should posts them to ``archive.ipython.org``.

You will need to use `twine <https://github.com/pypa/twine>`_ ) manually to
actually upload on PyPI. Unlike setuptools, twine is able to upload packages
over SSL.

    twine upload dist/*


PyPI/Warehouse will automatically hide previous releases. If you are uploading
a non-stable version, make sure to log-in to PyPI and un-hide previous version.


9. Draft a short release announcement
-------------------------------------

The announcement should include:

- release highlights
- a link to the html version of the *What's new* section of the documentation
- a link to upgrade or installation tips (if necessary)

Post the announcement to the mailing list and or blog, and link from Twitter.

.. note::

    If you are doing a RC or Beta, you can likely skip the next steps.

10. Update milestones on GitHub
-------------------------------

These steps will bring milestones up to date:

- close the just released milestone
- open a new milestone for the next release (x, y+1), if the milestone doesn't
  exist already

11. Update the IPython website
------------------------------

The IPython website should document the new release:

- add release announcement (news, announcements)
- update current version and download links
- update links on the documentation page (especially if a major release)

12. Update readthedocs
----------------------

Make sure to update readthedocs and set the latest tag as stable, as well as
checking that previous release is still building under its own tag. 


13. Celebrate!
--------------

Celebrate the release and please thank the contributors for their work. Great
job!

