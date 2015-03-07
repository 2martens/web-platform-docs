.. index::
    single: ACP Layout

ACP Layout
==========

The ACP layout consists not only of templates but also of the way the various
parts play together to form the visual unified experience called ACP. This
page has two main sections. The first one will elaborate on the general idea
and how the different parts play together, while the second part will be
focused on the technical aspect of the used variables, blocks, etc.

General Design
--------------

The core principle of the ACP is: keep it simple. After the initial setup
of a web project the ACP will be visited rarely (in comparison to the
frontend) and if it is visited the actions have to be performed quickly.

As a consequence of that the layout of the ACP should keep it as simple as
possible while still providing all needed functionality. Bad examples are
ACPs of widely used CMS applications like Joomla. Their ACP is full of
stuff which makes it hard to quickly find the relevant page.

To circumvent this problem the navigation of the 2WP ACP will be
straightforward and fast. There are only five top menu entries in a
Bootstrap-styled menu: Home, System, User Management, Appearance and Content.

Home will serve as the dashboard of the ACP, giving you all the relevant
information at a glance. The remaining four entries are the big categories
of actions and pages in the ACP. System contains all the things that
change the state of the system (the project) itself. Things like package
management or configuration belong in this area. User Management will contain
all the things that have to do with users and groups. Appearance deals with
the visual appearance of the website. Style management, menu management and
similar things will be found here. Last but not least comes the content area.
It will be the most dynamic of all since the content management happens here.
Whether you manage a forum structure, edit contents in a CMS or review
attachments of all kinds, the content area will be the place you spend most
of your ACP time in.

Each of the four areas will serve as a hub for all things beneath it. They
will contain buttons with icons on them (the technical side is explained
in the second section) that symbolize each an entire feature page. So the
configuration feature will most likely provide a button with a cog on it
that leads you to the configuration subpage. However each feature decides
alone how the main content area is structured. Therefore it is not possible
to say in advance how these subpages will look like. Some might use tabs,
because they have multiple subpages themselves, whereas others just have
one page.

Technical implementation
------------------------

This section will tell you what blocks are offered, what variables must be
provided and all other information that you need to develop subpages
for the ACP.

Variables
~~~~~~~~~

The layout requires the following variables from each subpage controller.
They are listed with name, possible values and function.

==================== =================================== =========================================
Name                 Values                              Function
==================== =================================== =========================================
navigation.active    home,system,user,appearance,content active navigation area
siteTitle            <String>                            content of <title>
area.title           <String>                            title of subpage
area.showBreadcrumbs true,false                          True on all subpages but the hub
breadcrumbs          <breadcrumb>                        array of breadcrumb arrays
<breadcrumb>.active  true,false                          True for active breadcrumb (the last one)
<breadcrumb>.path    <String>                            Route for breadcrumb
<breadcrumb>.title   <String>                            Translated title of breadcrumb
==================== =================================== =========================================

Blocks
~~~~~~

The layout offers the following blocks. They are listed by name and function.

=============== ===================
Name            Function
=============== ===================
innerContent    content of subpage
metaInformation all meta tags
stylesheets     loading stylesheets
header          common content
mainContent     main content
footer          common content
javascripts     loading javascript
=============== ===================
