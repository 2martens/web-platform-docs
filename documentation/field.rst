.. index::
    single: Field classes

Field classes
=============

What are Field classes and what are they used for? This article will answer
these questions. Let's start with a problem of PHP and how the Field
classes are the solution.

Problem
-------

PHP supports type hinting only for class types and arrays. This is
problematic if you handle large projects. Of course you can enforce
a policy of "one variable one type" and specify the type in the doc
comment. But in the end you still have to check the type when 3rd Party
code interacts with your code. Of course you can check the 3rd Party code
beforehand so that it calls the methods with the correct types. But this
is a lot of work.

And all the code to check the type is unwanted overhead as well. So
what can you do? Put everything inside an array? Well ... that's not a
real solution. The only way is to encapsulate the values inside class
types as these can be enforced by the language itself.

Solution
--------

Here the Field classes enter the stage. The AbstractField class implements
all the logic necessary to enforce the type of the field value. The
other classes like IntegerField or StringField don't really implement
something of their own. They only override the setValue and getValue
methods to provide a more specific doc comment containing their type.

Furthermore they expect only one argument in the constructor: The value.
Internally they are forwarding that value to the AbstractField constructor
and provide their type.

These classes make primitive type hinting possible. But the AbstractField
class is far more powerful. It also accepts arrays and class types as
values but that functionality probably won't be needed as much.

Usage
-----

The idea behind the Field classes is that you get a primitive value
from the user (e.g. through a form) and directly after validation it
ends up in one field object. This field object is then passed through
the application. With the type hinting and a unified interface in place,
it is easy to interact with that value and manipulate it when necessary.

Example:

.. code-block:: php

    // src/AppBundle/Controller/UserProfileController.php
    use TwoMartens\Bundle\CoreBundle\Field\StringField;

    // ...

    /**
     * Saves the form.
     */
    public function saveAction()
    {
        // ...
        $username = new StringField($username);
        // ...
    }

.. code-block:: php

    // src/AppBundle/User/UserService.php
    use TwoMartens\Bundle\CoreBundle\Field\StringField;

    // ...

    /**
     * Adds a user.
     *
     * @param StringField $username
     */
    public function addUser(StringField $username)
    {
        $userData = array(
            'username' => $username->getValue()
        );
        // ...
    }

Extendability
-------------

The IntegerField and StringField are just boxing classes for their
respective primitive types. But the Field classes allow for more complex
types as well. Imagine a birthday field. It has three internal fields:
Day, month and year. Each of them is an IntegerField but the BirthdayField
class adds additional checks to ensure that all of the three fields have
valid number ranges (e.g. month between 1 and 12 inclusive). Furthermore
the BirthdayField class provides a method to get the unix timestamp for
the date represented by the three values.

If the user enters a birthday in a form, these values can be directly
saved inside such a birthday field. This field object is then passed
around in the application and at any given point the user of the object
can be sure that it contains a valid value. This makes the job of
persistence easier as well, since you won't need checks for validity
anymore.
