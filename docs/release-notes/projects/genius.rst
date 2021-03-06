========================================================
Genius (Generic Network Interface, Utilities & Services)
========================================================

Genius project provides Generic Network Interfaces, Utilities & Services. Any
ODL application can use these to achieve interference-free co-existence with
other applications using Genius. OpendayLight Carbon Genius provides following
modules --

* **Interface (logical port) Manager** allows bindings/registration of
  multiple services to logical ports/interfaces
* **Overlay Tunnel Manager** creates and maintains overlay tunnels between
  configured tunnel endpoints
* **Aliveness Monitor** provides tunnel/nexthop aliveness monitoring services
* **ID Manager** generates cluster-wide persistent unique integer IDs
* **MD-SAL Utils** provides common generic APIs for interaction with MD-SAL
* **Resource Manager** provides a resource sharing framework for applications
  sharing common resources e.g. table-ids, group-ids etc.
* **FCAPS Application**  generates various alarms and counters for the different
  genius modules
* **FCAPS Framework**  module collectively fetches all data generated by fcaps
  application. Any underlying infrastructure can subscribe for its events to
  have a generic overview of the various alarms and counters

Major Features
==============

* **Features URL:** https://git.opendaylight.org/gerrit/gitweb?p=genius.git;a=blob_plain;f=features/genius-features/pom.xml

odl-genius
----------

* **Feature Description:**  This feature provides all functionalities provided by
  genius modules, including interface manager, tunnel manager, resource manager
  and ID manager and MDSAL Utils. It includes Genius APIs and implementation.

* **Top Level:** Yes
* **User Facing:** Yes
* **Experimental:** No
* **CSIT Tests:**
  * https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-1node-upstream-all-nitrogen/
  * https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-3node-upstream-all-nitrogen/

odl-genius-rest
---------------

* **Feature Description:**  This feature includes RESTCONF with 'odl-genius'
  feature.

* **Top Level:** Yes
* **User Facing:** Yes
* **Experimental:** No
* **CSIT Tests:**
  * https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-1node-upstream-all-nitrogen/
  * https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-3node-upstream-all-nitrogen/

odl-genius-api
---------------

* **Feature Description:**  This feature includes API for all the functionalities
  provided by Genius.

* **Top Level:** No
* **User Facing:** No
* **Experimental:** No
* **CSIT Tests:**
  * https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-1node-upstream-all-nitrogen/
  * https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-3node-upstream-all-nitrogen/

odl-genius-fcaps-application
----------------------------

* **Feature Description:**  includes genius FCAPS application.
* **Top Level:** Yes
* **User Facing:** Yes
* **Experimental:** Yes
* **CSIT Tests:** None

odl-genius-fcaps-framework
--------------------------

* **Feature Description:**  includes genius FCAPS Framework.
* **Top Level:** Yes
* **User Facing:** Yes
* **Experimental:** Yes
* **CSIT Tests:** None


New capabilities and enhancements added in Nitrogen
===================================================

Planned new capability added
----------------------------

* :doc:`/submodules/genius/docs/specs/service-recovery`


Enhancements added to project
-----------------------------

#. Migration to Karaf4
#. Bug fixes


Documentation
=============

* **Installation Guide(s):**

  * N/A

* **User Guide(s):**

  * :doc:`User Guide </user-guide/genius-user-guide>`

* **Developer Guide(s):**

  * :doc:`Developer Guide </submodules/genius/docs/index>`

Security Considerations
=======================

* Do you have any external interfaces other than RESTCONF?

  * No

* Other security issues?

  * N/A

Quality Assurance
=================

* `Sonar Report <https://sonar.opendaylight.org/overview?id=64114>`_

* `CSIT Jobs <https://jenkins.opendaylight.org/releng/view/genius/job/genius-csit-1node-upstream-all-nitrogen//>`_

* `Netvirt CSIT for Genius patches <https://jenkins.opendaylight.org/releng/job/genius-patch-test-netvirt-nitrogen/>`_

* `Netvirt Cluster CSIT for Genius patches <https://jenkins.opendaylight.org/releng/job/genius-patch-test-cluster-netvirt-nitrogen/>`_

  .. note:: Genius is used extensively in NetVirt, so NetVirt's CSIT also
            provides confidence in genius.

* Other manual testing and QA information

  * N/A

* Testing methodology. How extensive was it? What should be expected to work?
  What hasn't been tested as much?

  * fcaps_framework and fcaps_application features hasn't been tested much.

Migration
---------

* Is it possible to migrate from the previous release? If so, how?

  * No. OpenDaylight doesn't support migration natively for applications that
    use datastore.

Compatibility
-------------

* Is this release compatible with the previous release?

  * Functionality is fully backwards compatible.

* Any API changes?

  * New API added for `service-recovery </submodules/genius/docs/specs/service-recovery>` feature

* Any configuration changes?

  * No

Bugs Fixed
----------

* List of bugs fixed since the previous release

  * `Fixed BUGS <https://bugs.opendaylight.org/buglist.cgi?chfieldfrom=2017-05-25&chfieldto=2017-08-09&list_id=78466&product=genius&query_format=advanced&resolution=FIXED>`_

Known Issues
------------

* List key known issues with workarounds

  * None

* `Open Bugs <https://bugs.opendaylight.org/buglist.cgi?chfieldfrom=2016-08-9&chfieldto=2017-05-25&list_id=78466&product=genius&query_format=advanced&bug_status=__open__>`_

End-of-life
===========

* List of features/APIs which are EOLed, deprecated, and/or removed in this
  release

  * N/A

Standards
=========

* List of standards implemented and to what extent

  * N/A

Release Mechanics
=================

* `Release plan <https://wiki.opendaylight.org/view/Genius:Nitrogen_Release_Plan>`_

* Describe any major shifts in release schedule from the release plan

  * N/A
