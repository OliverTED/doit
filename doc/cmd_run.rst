==============================
Command line interface - run
==============================

A general `doit` command goes like this:

.. code-block:: console

    $ doit [run] [<options>] [<task|target> <task_options>]* [<variables>]


The `doit` command line contains several sub-commands. Most of the time you just
want to execute your tasks, that's what *run* does. Since it is by far the most
common operation it is also the default, so if you don't specify any sub-command
to *doit* it will execute *run*. So ``$ doit`` and ``$ doit run`` do the same
thing.

The basics of task selection were introduced in :ref:`Task Selection
<task-selection>`.


`python -m doit`
-----------------

`doit` can also be executed without using the `doit` script.

.. code-block:: console

   $ python -m doit

This is specially useful when testing `doit` with different python versions.


dodo file
----------

By default all commands are relative to ``dodo.py`` in the current folder.
You can specify a different *dodo* file containing task with the flag ``-f``.
This flag is valid for all sub-commands.


.. code-block:: console

    $ doit -f release.py


*doit* can seek for the ``dodo.py`` file on parent folders if the the option ``--seek-file`` is specified.


as an executable file
-----------------------

using a hashbang
^^^^^^^^^^^^^^^^^^^^^

If you have `doit` installed on ``/usr/bin`` use the following hashbang:

.. code-block:: bash

   #! /usr/bin/doit -f



using the API
^^^^^^^^^^^^^^

It is possible to make a ``dodo`` file become an executable on its own
by calling the ``doit.run()``, you need to pass the ``globals``:


.. literalinclude:: tutorial/executable.py

.. note::

  The ``doit.run()`` method will call ``sys.exit()`` so any code after it
  will not be executed.


``doit.run()`` parameter will be passed to a :ref:`ModuleTaskLoader <ModuleTaskLoader>` to find your tasks.



returned value
------------------

``doit`` process returns:

 * 0 => all tasks executed successfully
 * 1 => task failed
 * 2 => error executing task
 * 3 => error before task execution starts
        (in this case the reporter is not used)



config
--------

Command line parameters can be set straight on a `dodo` file. This example below sets the default tasks to be run, the ``continue`` option, and a different reporter.

.. literalinclude:: tutorial/doit_config.py

So if you just execute

.. code-block:: console

   $ doit

it will have the same effect as executing

.. code-block:: console

   $ doit --continue --reporter json my_task_1 my_task_2

You need to check `doit_cmd.py
<https://github.com/pydoit/doit/blob/master/doit/doit_cmd.py>`_ to find out how
parameter maps to config names.

.. note::

  The parameters ``--file`` and ``--dir`` can not be used on config because
  they control how the dodo file itself is loaded.


DB backend
--------------

`doit` saves the results of your tasks runs in a "DB-file", it supports
different backends:

 - `dbm`: (default) It uses `python dbm module <https://docs.python.org/3/library/dbm.html>`_. The actual DBM used depends on what is available on your machine/platform.

 - `json`: Plain text using a json structure, it is slow but good for debugging.

 - `sqlite3`: (experimental) very slow implementation, support concurrent access.


From the command line you can select the backend using the ``--backend`` option.

It is quite easy to add a new backend for any key-value store.


DB-file
----------

Option ``--db-file`` sets the name of the file to save the "DB",
default is ``.doit.db``.
Note that DBM backends might save more than one file, in this case
the specified name is used as a base name.

To configure in a `dodo` file the field name is ``dep_file``

.. code-block:: python

    DOIT_CONFIG = {
        'backend': 'json',
        'dep_file': 'doit-db.json',
    }


.. _verbosity_option:

verbosity
-----------

Option to change the default global task :ref:`verbosity<verbosity>` value.

.. code-block:: console

    $ doit --verbosity 2



dir (cwd)
-----------

By default the directory of the `dodo` file is used as the
"current working directory" on python execution.
You can specify a different *cwd* with the *-d*/*--dir* option.

.. code-block:: console

    $ doit --dir path/to/another/cwd


continue
---------

By default the execution of tasks is halted on the first task failure or error. You can force it to continue execution with the option --continue/-c

.. code-block:: console

    $ doit --continue


dry-run
--------

To quickly see which tasks might have to be rebuild, the option ``--dry-run`` disables task execution. The output shows which tasks that possibly need to be updated. 

.. code-block:: console

    $ doit --dry-run

Note: Without actual execution doit is unable to know which target contents will change. Dry run assumes that all targets are changed and outputs a list of possible executed tasks.

single task execution
----------------------

The option ``-s/--single`` can be used to execute a task without executing
its task dependencies.

.. code-block:: console

    $ doit -s do_something



.. _parallel-execution:

parallel execution
-------------------

`doit` supports parallel execution --process/-n.
This allows different tasks to be run in parallel, as long any dependencies are met.
By default the `multiprocessing <http://docs.python.org/library/multiprocessing.html>`_
module is used.
So the same restrictions also apply to the use of multiprocessing in `doit`.

.. code-block:: console

    $ doit -n 3

You can also execute in parallel using threads by specifying the option
`--parallel-type/-P`.

.. code-block:: console

    $ doit -n 3 -P thread


.. note::

   The actions of a single task are always run sequentially;
   only tasks and sub-tasks are affected by the parallel execution option.

.. _reporter:

reporter
---------

`doit` provides different "*reporters*" to display running tasks info
on the console.
Use the option --reporter/-r to choose a reporter.
Apart from the default it also includes:

 * executed-only: Produces zero output if no task is executed
 * json: Output results in JSON format
 * zero: display only error messages (does not display info on tasks
   being executed/skipped). This is used when you only want to see
   the output generated by the tasks execution.

.. code-block:: console

    $ doit --reporter json


custom reporter
-----------------

It is possible to define your own custom reporter. Check the code on
`doit/reporter.py
<https://github.com/pydoit/doit/blob/master/doit/reporter.py>`_ ... It is easy
to get started by sub-classing the default reporter as shown below. The custom
reporter must be configured using DOIT_CONFIG dict.

.. literalinclude:: tutorial/custom_reporter.py


Note that the ``reporter`` have no control over the *real time* output
from a task while it is being executed,
this is controlled by the ``verbosity`` param.



output-file
------------

The option --output-file/-o let you output the result to a file.

.. code-block:: console

    $ doit --output-file result.txt


pdb
-------

If the option ``--pdb`` is used, a post-mortem debugger will be launched in case
of a unhandled exception while loading tasks.



get_initial_workdir
---------------------

When `doit` executes by default it will use the location of `dodo.py`
as the current working directory (unless --dir is specified).
The value of `doit.get_initial_workdir` will contain the path from where
`doit` was invoked from.

This can be used for example set which tasks will be executed:

.. literalinclude:: tutorial/initial_workdir.py


minversion
-------------

`minversion` can be used to specify the minimum/oldest `doit` version
that can be used with a `dodo.py` file.

For example if your `dodo.py` makes use of a feature added at `doit X`
and distribute it. If another user who tries this `dodo.py` with a version
older that `X`, doit will display an error warning the user to update `doit`.

.. code-block:: console

    DOIT_CONFIG = {
        'minversion': '0.24.0',
    }


.. note::

  This feature was added on `doit` 0.24.0.
  Older Versions will not check or display error messages.


