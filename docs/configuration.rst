.. _configuration:

=============
Configuration
=============

``sphinx-multiversion`` reads your Sphinx :file:`conf.py` file for configuration.
As usual, you can also override certain options by using ``-D var=value`` on the command line.

This is what the default configuration looks like:

.. code-block:: python

    # Whitelist pattern for tags (set to None to ignore all tags)
    smv_tag_whitelist = r'^.*$'

    # Whitelist pattern for branches (set to None to ignore all branches)
    smv_branch_whitelist = r'^.*$'

    # Whitelist pattern for remotes (set to None to use local branches only)
    smv_remote_whitelist = None

    # Pattern for released versions
    smv_released_pattern = r'^tags/.*$'

    # Format for versioned output directories inside the build directory
    smv_outputdir_format = '{ref.name}'

    # Determines whether remote or local git branches/tags are preferred if their output dirs conflict
    smv_prefer_remote_refs = False

You can override all of these values inside your :file:`conf.py`.

.. note::

    You can check which tags/branches are matched by running ``sphinx-multiversion`` with the ``--dump-metadata`` flag. Branches or tags that don't contain both the sphinx source directory and the :file:`conf.py` file will be skipped automatically.

Tag/Branch/Remote whitelists
============================

Tags, Branches and Remotes are included by `Regular Expressions <python_regex_>`_.
Here are some examples:

.. code-block:: python

    smv_tag_whitelist = r'^.*$'                   # Include all tags
    smv_tag_whitelist = r'^v\d+\.\d+$'            # Include tags like "v2.1"

    smv_branch_whitelist = r'^.*$'                # Include all branches
    smv_branch_whitelist = r'^(?!master).*$'      # Include all branches except "master"

    smv_remote_whitelist = None                   # Only use local branches
    smv_remote_whitelist = r'^.*$'                # Use branches from all remotes
    smv_remote_whitelist = r'^(origin|upstream)$' # Use branches from origin and upstream

.. note::

    To list values to match, you can use ``git branch``, ``git tag`` and ``git remote``.


Release Pattern
===============

A Regular Expression is used to determine if a version of the documentation has been released or if it's a development version.
To allow more flexibility, the regex is evaluated over the full refname.

Here are some examples:

.. code-block:: python

    smv_released_pattern = r'^tags/.*$'           # Tags only
    smv_released_pattern = r'^heads/\d+\.\d+$'    # Branches like "2.1"
    smv_released_pattern = r'^(tags/.*|heads/\d+\.\d+)$'           # Branches like "2.1" and all tags
    smv_released_pattern = r'^(heads|remotes/[^/]+)/(?!:master).*$' # Everything except master branch

.. note::

    To list all refnames , you can use:

    .. code-block:: bash

        git for-each-ref --format "%(refname)" | sed 's/^refs\///g'


Output Directory Format
=======================

Each version will be built into a seperate subdirectory of the Sphinx output directory.
The ``smv_outputdir_format`` setting determines the directory structure for the subdirectories. It is a new-style Python formatting string with two parameters - ``ref`` and ``config``.

Here are some examples:

.. code-block:: python

    smv_outputdir_format = '{ref.name}'        # Use the branch/tag name
    smv_outputdir_format = '{ref.commit}'      # Use the commit hash
    smv_outputdir_format = '{ref.commit:.7s}'  # Use the commit hash truncated to 7 characters
    smv_outputdir_format = '{ref.refname}'     # Use the full refname
    smv_outputdir_format = '{ref.source}/{ref.name}'      # Equivalent to the previous example
    smv_outputdir_format = 'versions/{config.release}'    # Use "versions" as parent directory and the "release" variable from conf.py
    smv_outputdir_format = '{config.version}/{ref.name}'  # Use the version from conf.py as parent directory and the branch/tag name as subdirectory


.. seealso::

    Have a look at `PyFormat <python_format_>`_ for information how to use new-style Python formatting.

.. _speed builds:

Speed Up Builds
===============

By default all tags and branches matched by smv_tag_whitelist and smv_branch_whitelist
are rebuilt from scratch. This means that CI will take longer as more versions of your
source are tagged.

This can be avoided by adding the following to conf.py

.. code-block:: python

    smv_rebuild_tags = False

This causes sphinx multiversion to only build for tags that do not already exist in
the target folder. Branches are always built because they are subject to change.

Links to all versions in smv_tag_whitelist and smv_branch_whitelist are
always included in the generated sidebar as these have been previously built.

Thus, if you whitelist master and all released tags, this will only build
the reference in the current commit.

This will result in a set of documentation pages that have links to master and to
previous releases. There will be no links to future versions in historical documentation
pages. This is acceptable because master will always have links to all versions and all
versions will have a link to master.


Overriding Configuration Variables
==================================

You can override configuration variables the same way as you're used to with ``sphinx-build``.

Since ``sphinx-multiversion`` copies the branch data into a temporary directory and builds them there while leaving the current working directory unchanged, relative paths in your :file:`conf.py` will refer to the path of the version *you're building from*, not the path of the version you are trying to build documentation for.

Sometimes it might be necessary to override the configured path via a command line overide.
``sphinx-multiversion`` allows you to insert placeholders into your override strings that will automatically be replaced with the correct value for the version you're building the documentation for.

Here's an example for the `exhale extension <exhale_>`_:

.. code-block:: python

    sphinx-multiversion docs build/html -D 'exhale_args.containmentFolder=${sourcedir}/api'

.. note::

    Make sure to enclose the override string in single quotes (``'``) to prevent the shell from treating it as an environment variable and replacing it before it's passed to ``sphinx-multiversion``.

.. note::

    To see a list of available placeholder names and their values for each version you can use the ``--dump-metadata`` flag.

.. _python_regex: https://docs.python.org/3/howto/regex.html
.. _python_format: https://pyformat.info/
.. _exhale: https://exhale.readthedocs.io/en/latest/
