.. _sdni-dev-guide:

ODL-SDNi Developer Guide
========================

Overview
--------

This project aims at enabling inter-SDN controller communication by
developing SDNi (Software Defined Networking interface) as an
application (ODL-SDNi App).

ODL-SDNi Architecture
---------------------

-  SDNi Aggregator: Northbound SDNi plugin acts as an aggregator for
   collecting network information such as topology, stats, host etc.
   This plugin can be evolving as per needs of network data requested to
   be shared across federated SDN controllers.

-  SDNi API: API view autogenerated and accessible through RESTCONF to
   fetch the aggregated information from the northbound plugin – SDNi
   aggregator.The RESTCONF protocol operates on a conceptual datastore
   defined with the YANG data modeling language.

-  SDNi Wrapper: SDNi BGP Wrapper will be responsible for the sharing
   and collecting information to/from federated controllers.

-  SDNi UI:This component displays the SDN controllers connected to each
   other.

SDNi Aggregator
---------------

-  SDNiAggregator connects with the Base Network Service Functions of
   the controller. Currently it is querying network topology through
   MD-SAL for creating SDNi network capability.

-  SDNiAggregator is customized to retrieve the host controller’s
   details, while running the controller in cluster mode. Rest of the
   northbound APIs of controller will retrieve the entire topology
   information of all the connected controllers.

-  The SDNiAggregator creates a topology structure.This structure is
   populated by the various network funtions.

SDNi API
--------

Topology and QoS data is fetched from SDNiAggregator through RESTCONF.

`http://${controlleripaddress}:8181/apidoc/explorer/index.html <http://${controlleripaddress}:8181/apidoc/explorer/index.html>`__

`http://${ipaddress}:8181/restconf/operations/opendaylight-sdni-topology-msg:getAllPeerTopology <http://${ipaddress}:8181/restconf/operations/opendaylight-sdni-topology-msg:getAllPeerTopology>`__

**Peer Topology Data:** Controller IP Address, Links, Nodes, Link
Bandwidths, MAC Address of switches, Latency, Host IP address.

`http://${ipaddress}:8181/restconf/operations/opendaylight-sdni-qos-msg:get-all-node-connectors-statistics <http://${ipaddress}:8181/restconf/operations/opendaylight-sdni-qos-msg:get-all-node-connectors-statistics>`__

**QOS Data:** Node, Port, Transmit Packets, Receive Packets, Collision
Count, Receive Frame Error, Receive Over Run Error, Receive Crc Error

`http://${ipaddress}:8181/restconf/operations/opendaylight-sdni-qos-msg:get-all-peer-node-connectors-statistics <http://${ipaddress}:8181/restconf/operations/opendaylight-sdni-qos-msg:get-all-peer-node-connectors-statistics>`__

**Peer QOS Data:** Node, Port, Transmit Packets, Receive Packets,
Collision Count, Receive Frame Error, Receive Over Run Error, Receive
Crc Error

SDNi Wrapper
------------

.. figure:: ./images/SDNiWrapper.png
   :alt: SDNiWrapper

   SDNiWrapper

-  SDNiWrapper is an extension of ODL-BGPCEP where SDNi topology data is
   exchange along with the Update NLRI message. Refer
   http://tools.ietf.org/html/draft-ietf-idr-ls-distribution-04 for more
   information on NLRI.

-  SDNiWrapper gets the controller’s network capabilities through SDNi
   Aggregator and serialize it in Update NLRI message. This NLRI message
   will get exchange between the clustered controllers through
   BGP-UPDATE message. Similarly peer controller’s UPDATE message is
   received and unpacked then format to SDNi Network capability data,
   which will be stored for further purpose.

SDNi UI
-------

This component displays the SDN controllers connected to each other.

http://localhost:8181/index.html#/sdniUI/sdnController

API Reference Documentation
---------------------------

Go to
`http://${controlleripaddress}:8181/apidoc/explorer/index.html <http://${controlleripaddress}:8181/apidoc/explorer/index.html>`__,
sign in, and expand the opendaylight-sdni panel. From there, users can
execute various API calls to test their SDNi deployment.
