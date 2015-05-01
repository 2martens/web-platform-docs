.. index::
    single: ACP Dashboard

ACP Dashboard
=============

The ACP dashboard is the first page you will see in the ACP. It must
provide you with all the relevant information at a glance. This article
will explain how the dashboard is structured and how you can add your
own block.

Structure
---------

The dashboard contains several blocks. Each block is self-contained and
gives information about a particular topic. The blocks aren't hardcoded
but included via events. Therefore it is trivial for 3rd party developers
to include their own dashboard blocks for their bundles. All you need
is an event listener.

In the following section you can see a minimal working example.

Example
-------

First you need an event listener:

.. code-block:: php

    // src/AppBundle/EventListener/GoodMorningDashboardListener.php
    use Sonata\BlockBundle\Event\BlockEvent;
    use Sonata\BlockBundle\Model\Block;

    // ...

    /**
     * @param BlockEvent $event
     */
    public function onBlock(BlockEvent $event)
    {
        $content = '<p class="info">Good Morning</p>';

        $block = new Block();
        $block->setId(uniqid());
        $block->setSetting('content', $content);
        $block->setType('sonata.block.service.text');

        $event->addBlock($block);
    }

Second you need to configure it properly as a service:

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml
        services:
            good_morning.listener:
                class: AppBundle\EventListener\GoodMorningDashboardListener
                tags:
                    - { name: kernel.event_listener,
                        event: 'sonata.block.event.acp.dashboard',
                        method: onBlock }

    .. code-block:: xml

        <!-- app/config/services.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd"
        >
            <services>
                <service id="good_morning.listener"
                     class="AppBundle\EventListener\GoodMorningDashboardListener"
                 >
                    <tag name="kernel.event_listener"
                        event="sonata.block.event.acp.dashboard"
                        method="onBlock" />
                </service>
            </services>
        </container>

    .. code-block:: php

        // app/config/services.php
        $container
            ->register(
                'good_morning.listener',
                'AppBundle\EventListener\GoodMorningDashboardListener'
            )
            ->addTag(
                'kernel.event_listener',
                array(
                    'event' => 'sonata.block.event.acp.dashboard',
                    'method' => 'onBlock'
                )
            )
        ;

That's it. No more code needed to add your own block to the dashboard.
Of course this is just a minimal example. You can extend it quite heavily
by rendering a template inside the event listener and use the result
as the content. Just remember that the content of the block should use
Bootstrap for styling and nothing else.
