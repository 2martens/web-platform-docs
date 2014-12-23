.. index::
    single: Development

Development
===========

This page will focus on telling you how to contribute with development.
You will learn to set up your development environment, create pull requests
and work on new features or bugfixes.

Set up Development Environment
------------------------------

Choose the editor
~~~~~~~~~~~~~~~~~

Every developer needs a way to edit code. Since code is essentially text, a
simple text editor would work technically. But the probability to make
stupid mistakes is far higher. So the next bet are simple code editors like
Notepad++, These are not bad but they lack some crucial features that only
IDEs provide (e.g. easily run tests, see all project files at a glance).

OK, we need an IDE then. But which IDE is suited best to our situation? Well,
there is no one best solution. Take the IDE you can work with best.

You have an IDE now. The next step is to add the files created by the IDE to
your global .gitignore file. That way Git will ignore them in every Git repository
on your computer.

Set up Git
~~~~~~~~~~

Never heard of Git before? You should change this right now, before going on.
The `Git book`_ offers a good starting point for that. Next you have to create
forks of the official 2martens Web Platform repositories. These are hosted on `GitHub`_,
so you will need a GitHub account. GitHub should offer enough help pages to let you
set up your local clones of your forks.

As part of these steps you have to give your name and your email address. Be so kind
and use your real name here and the email address used on GitHub. We won't accept
contributions from people with fake names here. You don't need to use your real name
for GitHub though.

Set up Vagrant
~~~~~~~~~~~~~~

The repository of the 2martens Web Platform Edition provides a Vagrant config
file. This means you can easily set up a working server environment to do integration
tests. By default the server listens to port 8080 and the files can be accessed from
your browser with localhost:8080.

To make use of Vagrant, you need to install a virtualization solution. The provided
vagrant file is configured with the Virtualbox software. Virtualbox is a free solution
available on Windows, Linux and Mac. After you have installed Virtualbox, all you have
to do is navigate to the repository of the 2martens Web Platform Edition and type in the
following into the console:

.. code-block:: bash

   $ vagrant up

On the first time Vagrant will download the image used for the virtual machine from the
internet. Therefore you have to be connected to the internet when executing this command.

For more information on Vagrant, refer to the `documentation`_ of Vagrant.

Download dependencies
~~~~~~~~~~~~~~~~~~~~~

The 2martens Web Platform requires 3rd party dependencies to work. To download these
dependencies, you need to use Composer:

.. code-block:: bash

   $ curl -s http://getcomposer.org/installer | php
   $ php composer.phar install

This will take a while to execute but you should see the progress in the console.
After this command finishes, you have a working development environment.

Summary
~~~~~~~

Let's recap what you have done:

* installed an IDE
* set up Git and GitHub
* set up Vagrant
* downloaded dependencies

Work on new features or bugfixes
--------------------------------

Branch and commit strategy
~~~~~~~~~~~~~~~~~~~~~~~~~~

Don't just start to develop new features. Always look at the issue tracker first
and only pick those issues that are open and not yet assigned to somebody. If you
have found an issue and want to work on it, leave a notice in the issue. Next you
should checkout the appropriate branch on your local clone. For new features this
is master and for bugfixes the oldest still maintained branch in which the bug occurs.
Then create a new topic branch with the name ticket_<issueNr>.

Here is an example for a new feature and an issue with ID 10:

.. code-block:: bash

   $ git checkout master
   $ git checkout -b ticket_10

Inside this topic branch all the development for the changes related to the issue
is happening. Follow the principle: Commit often and early. Later on you can still
clean up the commit log by leaving small commits out of history. But it is not possible
to split commits up. As a rule of thumb: If you don't commit multiple times within
4 hours of work, you are doing something wrong.

The commits should be logically separate actions. It helps to design the specifics
of the work before starting. Furthermore there are almost certainly already UML
diagrams associated with new features, so following them is greatly appreciated.
That doesn't mean however that they are immutable. If you discover during your work
that the feature can be realised in a better way, update the UML diagram and continue
implementing.

Once your work is done, evaluate your changes. Do they break backward compatibility?
If the answer is yes, execute the following commands:

.. code-block:: bash

   $ git checkout master
   $ git pull
   $ git checkout topic_10
   $ git rebase master

If the answer is no, replace master with feature-track. If any conflicts occur
during rebase, fix them and retest your changes. If everything checks out,
perform a forced push to your fork of the official repository:

.. code-block:: bash

   $ git push -f origin topic_10

Now you just need to get the changes into the official repository.
This is done with a pull request. For detailed information take a look
at the dedicated section on this page.

Testing
~~~~~~~

One fundamental part of development is testing. The 2martens Web Platform
uses the PHP testing framework `PHPUnit`_. Every class but controllers
must have a corresponding test class with sensible test cases. We cannot give a
comprehensive guide to good testing here, but as a rule of thumb all the edge
cases should be covered.

Furthermore you should test the coding style with Codesniffer. The 2martens
Web Platform uses PSR-2 as coding style, similar to the coding style of Symfony.
Despite your best intentions you will make mistakes in this area, so better test
for it right away. Some IDEs can integrate Codesniffer into their live syntax
checking, which shows coding style errors directly in the editor.

We advise the usage of PHP Mess Detector to detect messy programming. Especially
the unused code rules should be used.

On top of the testing performed by every individual developer, every pull request
and commit to the official repositories is checked by Travis. Travis uses the rules
defined in the .travis.yml file. If a pull request does not run through successful,
it won't be integrated.

Create pull requests
--------------------

Your environment is ready to go, but are you? Well, that's a didactic question.
Of course you aren't ready yet. When you work on changes, you want them to end
in the official repository. To make this happen, you need to follow a certain
process. This process is largely borrowed from the `Symfony`_ project. In particular
we use the code contribution `guidelines`_ of Symfony.

There are some differences though: We use an additional ongoing branch named
feature-track. This branch contains new features that are backward compatible.
Therefore pay attention to the base branch in the selection. If your feature
implementation breaks backward compatibility, add [BC BREAK] into the title
of the pull request (is mentioned in the Symfony contribution guideline).

If your pull request covers a new feature, don't forget to link the associated
pull request for the documentation (in the overview table).

The overview table is listed hereafter:

.. code-block:: text

   | Q             | A
   | ------------- | ---
   | Bug fix?      | [yes|no]
   | New feature?  | [yes|no]
   | BC breaks?    | [yes|no]
   | Deprecations? | [yes|no]
   | Tests pass?   | [yes|no]
   | Fixed tickets | [comma separated list of tickets fixed by the PR]
   | License       | MIT
   | Doc PR        | [The reference to the documentation PR if any]

.. _`Git book`: http://git-scm.com
.. _`GitHub`: https://github.com
.. _`documentation`: http://TODO
.. _`Symfony`: http://symfony.com
.. _`PHPUnit`: TODO
.. _`guidelines`: http://symfony.com/doc/current/contributing/code/index.html
