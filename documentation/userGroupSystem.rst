.. index::
	single: User and Group system

User and Group system
=====================

Every multi-user system needs some way to differentiate
the users by way of authentication. In addition not every user should
be allowed to do all things which adds the necessity of authorization.
This is the fundamental wisdom almost every modern web application
is applying. But the similarities end there. There are dozens of
solutions with different implementations of this wisdom.

The 2martens Web Platform uses both users and groups to realize these
fundamental requirements. By default groups in Symfony are only a
collection of roles (see `documentation`_). But the 2WP groups are so
much more than that.

The 2WP not only embraces the arbitrary model which means that the admin
should have the power to decide which user group can do what. It also
allows more option types than boolean (roles). Of course you need to
specify for every option type what is the 'best' value (more on that
later).

Enough with the introduction. In the upcoming sections this article
will explain how the system works and how you can extend it.

* :ref:`high-level-overview`
* :ref:`how-to-add-your-own-options`

.. _documentation: https://symfony.com/doc/master/bundles/FOSUserBundle/groups.html

.. _high-level-overview:

High-level overview
-------------------

The User and Group system is split up into two major components:

* users
* groups

The FOS User Bundle is utilized in the background to deal with the
Symfony requirements for authentication and authorization. The 2WP
basically implements its own layer on top of that and relays the
basic information to the FOS User Bundle implementation while handling
the additional features by itself.

For data junkies: Users and groups live in a many-to-many relationship.

ACP navigation
^^^^^^^^^^^^^^

In the ACP you can find the related functionality under 'User Management'.
The functionality itself is presented there under 'User system' for
user-related features and 'Group system' for the group-related features.
Of course you can add your own entries there by using the
:doc:`ACP </documentation/acpLayout>` extension events. In fact the entries
of this User and Group system are also added by event.

Option types
^^^^^^^^^^^^

The User and Group system uses so called OptionTypes to get the effective
option if a user is in multiple groups. Each OptionType implements
a :code:`getBestValue` method that returns the best value from two
given values. How does the system know which option has which type?
With the help of event listeners you can specify exactly this (technical
details later on this page).

The group component provides a :code:`GroupService` which - among other
things - allows you to get the effective option or simply get the option
value for a specific group. Most of the time you will only use the
:code:`getEffective` method.

The best thing is that you don't need to use the service if you just want
to check for authorization (asking for roles). You can use the default
Symfony checks for roles with the user objects and they will work,
because the 2WP transparently fulfills the Symfony purpose of groups
and stores all boolean option specifically as roles in the group
object.

The Security component of Symfony uses the :code:`getRoles`
method of the User object to compare the required role with the
granted ones. The FOS User bundle already implements this method
and takes all the roles of the groups the user is a member of into
account. That way the 2WP implementation behaves like expected from
Symfony while also allowing far more option types.

Forms
^^^^^

The last major part of the User and Group system are the form types.
They are a good example for combining default (and easy) form behaviour
with more complicated solutions. By default all fields are mapped
to a model. This works well for most types of forms but it doesn't
work for the 2WP group options.

They are categorized under user, mod and admin and each are is
represented by an option category. These categories are indeed existing
in the model class but not in the database. Instead the options (not the
rest of the model) are stored in YAML files (similar to the storage of
the system-wide options).

Since the options are not mapped it is not trivial to validate them.
The easy and default validation functionality works with mapped fields
but not with unmapped ones. The group options can't be mapped, because
the model class itself must remain flexible and can't hard-code the
available options. And the categories are non-trivial fields which
cannot be easily validated anyway, because it is not clear what
kind of options will be found in each.

Like always you can add your own options here.

Groups
^^^^^^

The very last thing are the groups themselves. Nothing major but some
fundamental concepts should be made clear. Every group has these
properties (ignoring the already discussed options and computed
properties like amount of users or roles):

* role name
* name
* is essential
* can be empty

What do these properties mean? The role name is a unique identifier
of a group in the entire installation and will be used for the group role
or the links in the ACP. What is a group role? Every group has its own
group role (e.g. ROLE_ADMIN for the role name ADMIN). This allows you
to demand membership in a specific group to do things.

The name is the visible name. For multilingual environments this can
also be a translation string. The group names are translated in the
domain :code:`TwoMartensGroups`. The 2WP will provide translations for
the default groups which are enough to run most community websites.

All system-provided groups are essential and cannot be empty. All groups
created through the ACP are non-essential and can be empty. An essential
group cannot be deleted in the ACP. You cannot remove the last user
from a group that cannot be empty.

These groups will be provided (listed by role name):

* ADMIN
* MOD
* USER
* GUEST
* EVERYONE

During initial installation one user will be created who will be
part of both the USER and ADMIN group. By default the ADMIN group
may do everything. Every user will be part of the USER group.
In the group EVERYONE you can find all users and additionally every
person who is not logged in. The GUEST group contains only people
who are not logged in.

.. _how-to-add-your-own-options:

How to add your own options?
----------------------------

It is surprisingly easy to add your own options and subsequently check
for them. You need these things:

* event listener to add your options to the group form (three events
  available)

    * :code:`twomartens.core.group_type.acp_options`
    * :code:`twomartens.core.group_type.mod_options`
    * :code:`twomartens.core.group_type.user_options`

* event listener for the :code:`twomartens.core.group_service.init`
  event (to provide mapping between options and option types)
* translations for your options in a translation file
* TBD (to add options when your bundle gets installed)
* TBD (to provide default values for your options)

The TBD values will be sorted out alongside the package system.

Let's start with the first event listener. Most things are already
abstracted away in the :code:`AbstractGroupOptionListener` which you
need to subclass. Here is a minimal example for one boolean option in the
ACP super category:

.. code-block:: php

    // src/AppBundle/EventListener/ACPGroupOptionListener.php
    use Symfony\Component\Form\ChoiceList\ChoiceListInterface;

    // ...

    /**
     * Maps option name to form field type.
     * @var string[]
     */
    private $fieldMap;

    /**
     * Maps option name to multiple setting.
     * @var boolean[]
     */
    private $multipleMap;

    /**
     * Maps option name to choice list.
     * @var ChoiceListInterface[]
     */
    private $choicesMap;

    /**
     * {@inheritdoc}
     */
    public function __construct()
    {
        $this->fieldMap = [
            'forum_add' => 'checkbox'
        ];
        $this->multipleMap = [];
        $this->choicesMap = [];
    }

    /**
     * {@inheritdoc}
     */
    protected function getLabelPrefix()
    {
        return parent::getLabelPrefix().'.acp';
    }

    /**
     * Returns the name of the category this listener is responsible for.
     *
     * @return string
     */
    protected function getCategoryName()
    {
        return 'app';
    }

    /**
     * Returns the field map.
     *
     * The field map maps the option name to the form field type.
     *
     * @return string[]
     */
    protected function getFieldMap()
    {
        return $this->fieldMap;
    }

    /**
     * Returns the multiple map.
     *
     * The multiple map maps the option name to the value of the multiple
     * setting. Only relevant for choice fields.
     *
     * @return boolean[]
     */
    protected function getMultipleMap()
    {
        $this->multipleMap;
    }

    /**
     * Returns the choice list map.
     *
     * The choice list map maps the option name to the corresponding
     * choice list. Only relevant for choice fields.
     *
     * @return ChoiceListInterface[]
     */
    protected function getChoiceListMap()
    {
        return $this->choicesMap;
    }

    /**
     * Returns the translation domain.
     *
     * @return string
     */
    protected function getDomain()
    {
        return 'AppBundle';
    }

This piece of code ensures that the option is properly rendered.
You don't exactly say which option type the option has but rather
how it should be displayed. An integer option might use a text field
or a select or radio buttons to determine the integer value.

Next up is the configuration to make it known to the event component.

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml
        services:
            option.listener:
                class: AppBundle\EventListener\ACPGroupOptionListener
                tags:
                    - { name: kernel.event_listener,
                        event: 'twomartens.core.group_type.acp_options',
                        method: onBuildForm }

    .. code-block:: xml

        <!-- app/config/services.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd"
        >
            <services>
                <service id="option.listener"
                     class="AppBundle\EventListener\GroupOptionListener"
                >
                    <tag name="kernel.event_listener"
                        event="twomartens.core.group_type.acp_options"
                        method="onBuildForm" />
                </service>
            </services>
        </container>

    .. code-block:: php

        // app/config/services.php
        $container
            ->setDefinition(
                'option.listener',
                new Definition(
                    'AppBundle\EventListener\ACPGroupOptionListener'
                )
            )
            ->addTag(
                'kernel.event_listener',
                array(
                    'event' => 'twomartens.core.group_type.acp_options',
                    'method' => 'onBuildForm'
                )
            )
        ;

The last element is the translation file.

.. code-block:: xml

    // src/AppBundle/Resources/translations/AppBundle.en.xliff

    <?xml version="1.0"?>
    <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
        <file source-language="en" datatype="plaintext" original="" >
            <body>
                <trans-unit id="acp.group.options.acp.app.forum_add.label">
                    <source>acp.group.options.acp.app.forum_add.label</source>
                    <target>Can add forum boards</target>
                </trans-unit>
            </body>
        </file>
    </xliff>

For the other two categories you only need to replace :code:`acp` with
:code:`mod` or :code:`user`.
One thing is still missing: the mapping to the option type. For that
purpose you also need an event listener which would look like this (in
this example):

.. code-block:: php

    // src/AppBundle/EventListener/GroupOptionTypeListener.php
    use TwoMartens\Bundle\CoreBundle\Event\GroupOptionTypeEvent;
    use TwoMartens\Bundle\CoreBundle\Group\Option\BooleanOptionType;
    // ...

    /**
     * Called during initialization of the Group service.
     *
     * @param GroupOptionTypeEvent $event
     */
    public function onGroupServiceInit(GroupOptionTypeEvent $event)
    {
         $optionTypes = [
            'acp' => [
                'app' => [
                    'forum_add' => new BooleanOptionType()
                ]
            ]
        ];
        $event->addOptions($optionTypes);
    }

It's important to understand that this listener is used for ALL your
group options (all three categories combined). Obviously the event
component doesn't know about this listener yet. The configuration
would look like this:

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml
        services:
            option.listener:
                class: AppBundle\EventListener\GroupOptionTypeListener
                tags:
                    - { name: kernel.event_listener,
                        event: 'twomartens.core.group_service.init',
                        method: onGroupServiceInit }

    .. code-block:: xml

        <!-- app/config/services.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd"
        >
            <services>
                <service id="option.listener"
                     class="AppBundle\EventListener\GroupOptionTypeListener"
                >
                    <tag name="kernel.event_listener"
                        event="twomartens.core.group_service.init"
                        method="onGroupServiceInit" />
                </service>
            </services>
        </container>

    .. code-block:: php

        // app/config/services.php
        $container
            ->setDefinition(
                'option.listener',
                new Definition(
                    'AppBundle\EventListener\GroupOptionTypeListener'
                )
            )
            ->addTag(
                'kernel.event_listener',
                array(
                    'event' => 'twomartens.core.group_service.init',
                    'method' => 'onGroupServiceInit'
                )
            )
        ;

That are all steps needed to handle the daily work. What's missing is
the initial work to provide the system with the options. Without this
initial work all these things here are nice but without effect:
If the described options are not found in the option YAML files then
the corresponding listeners won't do anything.

On the other hand if options are found that are not described then
the behaviour is unclear. Exceptions or fatal errors may happen.
