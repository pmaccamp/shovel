Shovel
======

Shovel is like Rake for python. Turn python funcitons into tasks simply, and 
access and invoke them from the command line. 'Nuff said. __New__ Shovel also
now has support for invoking the same tasks in the browser you'd normally run
from the command line, without any modification to your shovel scripts.

Philosophy of Shovel
--------------------

- Tasks should be easy to define
- Making a function a task should change as little as possible
- Arguments are strings -- we're not going to try to guess (there's an exception)
- We value specificity
- We'll inspect your tasks as much as we can
- Tasks should be _accessible_

Installing Shovel
-----------------

Like most python projects, build with the included `setup.py`:

	python setup.py install

Currently the only dependency of shovel is the `argparse` module.

Using Shovel
------------

Shovel looks for a file in the current working directory called `shovel.py`
and executes it to find tasks you've defined (more on that in a second). If
you'd like to modularize your tasks, create a `shovel` directory, and put
as many python files as you'd like in that directory, and make as many files
as you'd like in there. Do this recursively if you'd like. For example:

	shovel/
		foo.py
		bar.py
		testing/
			foo.py
			zod.py
		util/
			hello.py

In this way, you can 'modularize' your tasks. By 'modularize,' these are not
full python modules (and we don't currently examing `__init__.py` in each 
directory), but it makes it convenient for organization. Each task is prepended
with the file and directory names. For example, if your `shovel/testing/foo.py`
defined a task `bar`, then that task would have the name `testing.foo.bar`.

If you define tasks in `shovel.py` instead of using a `shovel` directory, those
tasks will be in the global namespace. If `shovel.py` defines a task `bar`, then
that task's name would simply be `bar`.

In these python files, use shovel by importing shovel's task decorator. Then,
apply as needed:

	from shovel import task
	
	@task
	def hello(name):
		'''Prints hello and the provided name'''
		print 'Hello, %s' % name
	
	def not_a_task():
		'''Print I'm not considered a task in shovel'''
		pass

Command Line Utility
--------------------

Invoke shovel with the `shovel` command. If you would like to know more about
what functions that shovel knows about:

	shovel help

If you'd like more information on a specific task or module, you can ask for
more information with shovel help. Shovel can figure out lots of things about
your tasks. Their names, file, line number, arguments, default arguments, if
they take a variable number of parameters, and so forth. When you ask for help
on a specific task, everything we know about that task will be presented to you.

	# List the tasks in the testing directory
	shovel help testing
	# Get more help on the testing.test task
	shovel help testing.test

Execute tasks with shovel and then the task name

	shovel foo.hello

Arguments are passed in a strings, and we really try to give you the same
semantics as when you'd normally invoke a function in python. For example,
arguments are considered positional arguments by default, but you can provide
a keyword name for specificity. For example, to execute `foo.bar` in a way
equivalent to `foo('1', '2', '3', hello='7')`, you would invoke it:

	shovel foo.bar 1 2 3 --hello 7

Keyword names are merely stripped of the leading dashes when parsed. Also
be warned that shovel options (like `--verbose` and `--dry-run`) will __not__
be available to your function. Speaking of which, if you would like shovel
to be extra talkative (for debugging, perhaps), use the `--verbose` switch:

	shovel --verbose foo.bar 1 2 3 --hello 7

Shovel has a dry-run option that will accept all the parameters you would 
normally pass into a task, but merely tells you how it would invoke a task.
This can be helpful if you want to inspect the arguments that your task 
would get, to make sure that it's correctly invoked:

	shovel --dry-run foo.bar 1 2 3 --hello 7

The one exception to arguments not being interpreted as strings is that 
orphan keyword arguments are interpreted as flags meaning 'True.' For example,
if we executed the following, then `a` and `b` would be passed as True:

	shovel foo.bar --a --b

The reason for this is that flags are common for tasks, and it's a relatively
unambiguous syntax. To a human, the meaning is clear, and now it is to shovel.

Browser
-------

Shovel also now comes with a small utility using the [`bottle`](http://bottlepy.org/docs/dev/)
framework, designed to make all your shovel tasks accessible from a browser.
At some point, I'd like to make it accessible as an API as well, returning 
JSON blobs instead of HTML output when requested. That said, it's not a high
priority -- if it's something you're after, let me know!

You can access the browser utility by starting up the `shovel-server` utility
from the same directory where you'd normally run your shovel tasks. You may 
optionally supply the `--port` option to specify the port on which you'd like
it to listen, and `--verbose` for additional output:

	# From the directory where your shovel tasks are
	shovel-server

By default, the `shovel-server` listens on port 3000, and you can access many
of the same utilities you would from the command line. For instance, help is
available through the [/help](http://localhost:3000/help) endpoint. Help for
a specific function is available by providing the name of the task (or group)
as a query parameter. To get more help on task `foo.bar`, you'd visit
[/help?foo.bar](http://localhost:3000/help?foo.bar), etc.

Tasks are executed by visiting the `/<task-name>` end-point, and the query 
parameters are what gets provided to the function. Query parameters without
values are considered to be positional, and query parameters with values are
considered to be keyword arguments. For example, the following are equivalent:

	# Accessing through the HTTP interface
	curl http://localhost:3000/foo.bar?hello&and&how&are=you
	# Accessing through the command-line utility
	shovel foo.bar hello and how --are=you
	# Executing the original python function
	...
	>>> bar('hello', 'and', 'how', are='you')

In this way, we can support conventional arguments, variable arguments, and
keyword arguments. That said, there is a slight difference in the invocation
from the command-line and through the browser. In the command-line tool, 
keyword arguments without values are interpreted as flags, where in the url,
that is not the case. For example, in the following invocation, both 'a' and
'b' would be passed as 'True' into the function, but there is no equivalent
in the URL form:

	shovel foo.bar --a --b

A convenient feature of the shovel server is that it checks the last-modified
time of the input shovel files, and reimports any definitions that have been
updated since it last checked. So if you save your shovel files, the changes
will be reflected in the web app.

To-Do
=====

1. Allow the `shovel-server` utility to return `application/json` when requested