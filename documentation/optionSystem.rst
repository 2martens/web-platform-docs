.. index::
	single: Option system

Option system
=============

What is this Option system? What are options? This article will
answer these questions. It will start with the options itself and
continues with the system afterwards.

Options
-------

The options are somewhat the settings of the 2martens Web Platform
and all installed applications. You can edit them in the ACP. They
should not be confused with the options you can configure in the
config files (config.yml, config_<env>.yml).

The options can be used by applications to allow the administrator
to customize the functionality of the application. They support
the following value types:

* boolean
* integer
* float
* array
* string

These value types can be translated to various form field types.
That said, let's come to the system part of it and how you can
add your own options.

Provide options for your bundle
-------------------------------

In order to provide options for your bundle, you need to do some things:

* event listener for the option form
* translation messages for the displayed labels of the options
* default options file for your bundle (specifics: TBD)

What form? The Option system handles almost the entire thing: reading options
from the persistence file (options.yml), building a form (fires event),
validating the submitted form and saving the changes back to the persistence
file.

In this chain of commands there is one step where you need to plug in with
your code: building a form. The system can read the options but it cannot
know how you want to ask the admin to enter the options. An array field can
be filled by a multi-select or multiple checkboxes. A single value (integer,
float, string) could be filled by an input field, a select field or multiple
radio buttons. Therefore it is necessary that you specify how each of the options
of your bundle has to be rendered in the option form.

The event listener does exactly that. The Option system provides an abstract event
listener for that purpose to minimize coding effort and redundancy. The abstract
class is named :code:`AbstractOptionListener` in the namespace
:code:`\TwoMartens\Bundle\CoreBundle\EventListener\AbstractOptionListener` and
already implements all the heavy lifting. It requires some information though,
which you have to provide by implementing the abstract methods of said class.

An example is always better than anything, so here it comes:

.. code-block:: php

    // src/AppBundle/EventListener/OptionListener.php
    use Symfony\Component\Form\ChoiceList\ChoiceListInterface;
    use Symfony\Component\Translation\TranslatorInterface;

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
    public function __construct(TranslatorInterface $translator)
    {
        parent::__construct($translator);
        $this->fieldMap = [
            'adminEmail' => 'email'
        ];
        $this->multipleMap = [];
        $this->choicesMap = [];
    }

    /**
     * {@inheritdoc}
     */
    protected function getCategoryName()
    {
        return 'app';
    }

    /**
     * {@inheritdoc}
     */
    protected function getFieldMap()
    {
        return $this->fieldMap;
    }

    /**
     * {@inheritdoc}
     */
    protected function getMultipleMap()
    {
        return $this->multipleMap;
    }

    /**
     * {@inheritdoc}
     */
    protected function getChoiceListMap()
    {
        return $this->choicesMap;
    }

    /**
     * {@inheritdoc}
     */
    protected function getDomain()
    {
        return 'AppBundle';
    }

With this code you will ensure that the option (should it appear in the options.yml) is properly
rendered in the form. You still need to configure the listener though to make it known to the event
component.

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml
        services:
            option.listener:
                class: AppBundle\EventListener\OptionListener
                arguments: [@translator]
                tags:
                    - { name: kernel.event_listener,
                        event: 'twomartens.core.option_configuration.build_form',
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
                     class="AppBundle\EventListener\OptionListener"
                >
                    <argument type="service" id="translator"/>
                    <tag name="kernel.event_listener"
                        event="twomartens.core.option_configuration.build_form"
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
                    'AppBundle\EventListener\OptionListener',
                    array(new Reference('translator'))
                )
            )
            ->addTag(
                'kernel.event_listener',
                array(
                    'event' => 'twomartens.core.option_configuration.build_form',
                    'method' => 'onBuildForm'
                )
            )
        ;

The last thing you need is a translation file.

.. code-block:: xml

    // src/AppBundle/Resources/translations/AppBundle.en.xliff

    <?xml version="1.0"?>
    <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
        <file source-language="en" datatype="plaintext" original="" >
            <body>
                <trans-unit id="acp.options.app.adminEmail.label">
                    <source>acp.options.app.adminEmail.label</source>
                    <target>Administrator Email</target>
                </trans-unit>
            </body>
        </file>
    </xliff>

With this code you have everything set up to display your own options.
What's missing is the part during the installation to tell the 2martens
Web Platform what options are provided by your bundle. These options
will be added to the options.yml. The specifics of that are TBD though.
Until the Package system has been implemented, it is not possible to
explain this reliably.
