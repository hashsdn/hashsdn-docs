.. _sxp-dev-guide:

SXP Developer Guide
===================

Overview
--------

SXP (Scalable-Group Tag eXchange Protocol) project is an effort to enhance
OpenDaylight platform with IP-SGT (IP Address to Source Group Tag)
bindings that can be learned from connected SXP-aware network nodes. The
current implementation supports SXP protocol version 4 according to the
Smith, Kandula - SXP `IETF
draft <https://tools.ietf.org/html/draft-smith-kandula-sxp-05>`__ and
grouping of peers and creating filters based on ACL/Prefix-list syntax
for filtering outbound and inbound IP-SGT bindings. All protocol legacy
versions 1-3 are supported as well. Additionally, version 4 adds
bidirectional connection type as an extension of a unidirectional one.

SXP Architecture
----------------

The SXP Server manages all connected clients in separate threads and a
common SXP protocol agreement is used between connected peers. Each SXP
network peer is modelled with its pertaining class, e.g., SXP Server
represents the SXP Speaker, SXP Listener the Client. The server program
creates the ServerSocket object on a specified port and waits until a
client starts up and requests connect on the IP address and port of the
server. The client program opens a Socket that is connected to the
server running on the specified host IP address and port.

The SXP Listener maintains connection with its speaker peer. From an
opened channel pipeline, all incoming SXP messages are processed by
various handlers. Message must be decoded, parsed and validated.

The SXP Speaker is a counterpart to the SXP Listener. It maintains a
connection with its listener peer and sends composed messages.

The SXP Binding Handler extracts the IP-SGT binding from a message and
pulls it into the SXP-Database. If an error is detected during the
IP-SGT extraction, an appropriate error code and sub-code is selected
and an error message is sent back to the connected peer. All transitive
messages are routed directly to the output queue of SXP Binding
Dispatcher.

The SXP Binding Dispatcher represents a selector that will decides how
many data from the SXP-database will be sent and when. It is responsible
for message content composition based on maximum message length.

The SXP Binding Filters handles filtering of outgoing and incoming
IP-SGT bindings according to BGP filtering using ACL and Prefix List
syntax for specifying filter or based on Peer-sequence length.

The SXP Domains feature provides isolation of SXP peers and bindings
learned between them, also exchange of Bindings is possible across
SXP-Domains by ACL, Prefix List or Peer-Sequence filters

Key APIs and Interfaces
-----------------------

As this project is fairly small, it provides only few features that
install and provide all APIs and implementations for this project.

-  sxp-route

-  sxp-controller

-  sxp-api

-  spx-core

sxp-route
~~~~~~~~~

Performs managing of SXP devices in cluster environment

sxp-controller
~~~~~~~~~~~~~~

RPC request handling

sxp-api
~~~~~~~

Contains data holders and entities

spx-core
~~~~~~~~

Main logic and core features

API Reference Documentation
---------------------------

`RESTCONF Interface and Dynamic
Tree <https://wiki.opendaylight.org/images/9/91/SXP_Restconf_Interface_and_Dynamic_Tree.pdf>`__
`Specification and
Architecture <https://wiki.opendaylight.org/images/4/44/SXP_Specification_and_Architecture_v05.pdf>`__

