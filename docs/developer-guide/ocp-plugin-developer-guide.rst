.. _ocpplugin-dev-guide:

OCP Plugin Developer Guide
==========================

This document is intended for both OCP (ORI [Open Radio Interface] C&M
[Control and Management] Protocol) agent developers and OpenDaylight
service/application developers. It describes essential information
needed to implement an OCP agent that is capable of interoperating with
the OCP plugin running in OpenDaylight, including the OCP connection
establishment and state machines used on both ends of the connection. It
also provides a detailed description of the northbound/southbound APIs
that the OCP plugin exposes to allow automation and programmability.

Overview
--------

OCP is an ETSI standard protocol for control and management of Remote
Radio Head (RRH) equipment. The OCP Project addresses the need for a
southbound plugin that allows applications and controller services to
interact with RRHs using OCP. The OCP southbound plugin will allow
applications acting as a Radio Equipment Control (REC) to interact with
RRHs that support an OCP agent.

.. figure:: ./images/ocpplugin/ocp-sb-plugin.jpg
   :alt: OCP southbound plugin

   OCP southbound plugin

Architecture
------------

OCP is a vendor-neutral standard communications interface defined to
enable control and management between RE and REC of an ORI architecture.
The OCP Plugin supports the implementation of the OCP specification; it
is based on the Model Driven Service Abstraction Layer (MD-SAL)
architecture.

The OCP Plugin project consists of three main components: OCP southbound
plugin, OCP protocol library and OCP service. For details on each of
them, refer to the OCP Plugin User Guide.

.. figure:: ./images/ocpplugin/plugin-design.jpg
   :alt: Overall architecture

   Overall architecture

Connection Establishment
------------------------

The OCP layer is transported over a TCP/IP connection established
between the RE and the REC. OCP provides the following functions:

-  Control & Management of the RE by the REC

-  Transport of AISG/3GPP Iuant Layer 7 messages and alarms between REC
   and RE

Hello Message
~~~~~~~~~~~~~

Hello message is used by the OCP agent during connection setup. It is
used for version negotiation. When the connection is established, the
OCP agent immediately sends a Hello message with the version field set
to highest OCP version supported by itself, along with the verdor ID and
serial number of the radio head it is running on.

The combinaiton of the verdor ID and serial number will be used by the
OCP plugin to uniquely identify a managed radio head. When not receiving
reply from the OCP plugin, the OCP agent can resend Hello message with
pre-defined Hello timeout (THLO) and Hello resend times (NHLO).

According to ORI spec, the default value of TCP Link Monitoring Timer
(TTLM) is 50 seconds. The RE shall trigger an OCP layer restart while
TTLM expires in RE or the RE detects a TCP link failure. So we may
define NHLO \* THLO = 50 seconds (e.g. NHLO = 10, THLO = 5 seconds).

By nature the Hello message is a new type of indication, and it contains
supported OCP version, vendor ID and serial number as shown below.

**Hello message.**

::

    <?xml version="1.0" encoding="UTF-8"?>
    <msg xmlns="http://uri.etsi.org/ori/002-2/v4.1.1">
      <header>
        <msgType>IND</msgType>
        <msgUID>0</msgUID>
      </header>
      <body>
        <helloInd>
          <version>4.1.1</version>
          <vendorId>XYZ</vendorId>
          <serialNumber>ABC123</serialNumber>
        </helloInd>
      </body>
    </msg>

Ack Message
~~~~~~~~~~~

Hello from the OCP agent will always make the OCP plugin respond with
ACK. In case everything is OK, it will be ACK(OK). In case something is
wrong, it will be ACK(FAIL).

If the OCP agent receives ACK(OK), it goes to Established state. If the
OCP agent receives ACK(FAIL), it goes to Maintenance state. The failure
code and reason of ACK(FAIL) are defined as below:

-  FAIL\_OCP\_VERSION (OCP version not supported)

-  FAIL\_NO\_MORE\_CAPACITY (OCP plugin cannot control any more radio
   heads)

The result inside Ack message indicates OK or FAIL with different
reasons.

**Ack message.**

::

    <?xml version="1.0" encoding="UTF-8"?>
    <msg xmlns="http://uri.etsi.org/ori/002-2/v4.1.1">
      <header>
        <msgType>ACK</msgType>
        <msgUID>0</msgUID>
      </header>
      <body>
        <helloAck>
          <result>FAIL_OCP_VERSION</result>
        </helloAck>
      </body>
    </msg>

State Machines
~~~~~~~~~~~~~~

The following figures illustrate the Finite State Machine (FSM) of the
OCP agent and OCP plugin for new connection procedure.

.. figure:: ./images/ocpplugin/ocpagent-state-machine.jpg
   :alt: OCP agent state machine

   OCP agent state machine

.. figure:: ./images/ocpplugin/ocpplugin-state-machine.jpg
   :alt: OCP plugin state machine

   OCP plugin state machine

Northbound APIs
---------------

There are ten exposed northbound APIs: health-check, set-time, re-reset,
get-param, modify-param, create-obj, delete-obj, get-state, modify-state
and get-fault

health-check
~~~~~~~~~~~~

The Health Check procedure allows the application to verify that the OCP
layer is functioning correctly at the RE.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:health-check-nb

POST Input
^^^^^^^^^^

+--------------------+----------+--------------------+--------------------+----------+
| Field Name         | Type     | Description        | Example            | Required |
|                    |          |                    |                    | ?        |
+====================+==========+====================+====================+==========+
| nodeId             | String   | Inventory node     | ocp:MTI-101-200    | Yes      |
|                    |          | reference for OCP  |                    |          |
|                    |          | radio head         |                    |          |
+--------------------+----------+--------------------+--------------------+----------+
| tcpLinkMonTimeout  | unsigned | TCP Link           | 50                 | Yes      |
|                    | Short    | Monitoring Timeout |                    |          |
|                    |          | (unit: seconds)    |                    |          |
+--------------------+----------+--------------------+--------------------+----------+

**Example.**

::

    {
        "health-check-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "tcpLinkMonTimeout": "50"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| result             | String, enumerated | Common default result codes          |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "result": "SUCCESS"
        }
    }

set-time
~~~~~~~~

The Set Time procedure allows the application to set/update the absolute
time reference that shall be used by the RE.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:set-time-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| newTime    | dateTime   | New datetime setting | 2016-04-26T10:23:00- | Yes        |
|            |            | for radio head       | 05:00                |            |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "set-time-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "newTime": "2016-04-26T10:23:00-05:00"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| result             | String, enumerated | Common default result codes +        |
|                    |                    | FAIL\_INVALID\_TIMEDATA              |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "result": "SUCCESS"
        }
    }

re-reset
~~~~~~~~

The RE Reset procedure allows the application to reset a specific RE.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:re-reset-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "re-reset-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| result             | String, enumerated | Common default result codes          |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "result": "SUCCESS"
        }
    }

get-param
~~~~~~~~~

The Object Parameter Reporting procedure allows the application to
retrieve the following information:

1. the defined object types and instances within the Resource Model of
   the RE

2. the values of the parameters of the objects

Default URL:
http://localhost:8181/restconf/operations/ocp-service:get-param-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| objId      | String     | Object ID            | RxSigPath\_5G:1      | Yes        |
+------------+------------+----------------------+----------------------+------------+
| paramName  | String     | Parameter name       | dataLink             | Yes        |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "get-param-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "objId": "RxSigPath_5G:1",
                "paramName": "dataLink"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| id                 | String             | Object ID                            |
+--------------------+--------------------+--------------------------------------+
| name               | String             | Object parameter name                |
+--------------------+--------------------+--------------------------------------+
| value              | String             | Object parameter value               |
+--------------------+--------------------+--------------------------------------+
| result             | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_UNKNOWN\_PARAM"               |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "obj": [
                {
                    "id": "RxSigPath_5G:1",
                    "param": [
                        {
                            "name": "dataLink",
                            "value": "dataLink:1"
                        }
                    ]
                }
            ],
            "result": "SUCCESS"
        }
    }

modify-param
~~~~~~~~~~~~

The Object Parameter Modification procedure allows the application to
configure the values of the parameters of the objects identified by the
Resource Model.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:modify-param-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| objId      | String     | Object ID            | RxSigPath\_5G:1      | Yes        |
+------------+------------+----------------------+----------------------+------------+
| name       | String     | Object parameter     | dataLink             | Yes        |
|            |            | name                 |                      |            |
+------------+------------+----------------------+----------------------+------------+
| value      | String     | Object parameter     | dataLink:1           | Yes        |
|            |            | value                |                      |            |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "modify-param-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "objId": "RxSigPath_5G:1",
                "param": [
                    {
                        "name": "dataLink",
                        "value": "dataLink:1"
                    }
                ]
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| objId              | String             | Object ID                            |
+--------------------+--------------------+--------------------------------------+
| globResult         | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_PARAMETER\_FAIL",             |
|                    |                    | "FAIL\_NOSUCH\_RESOURCE"             |
+--------------------+--------------------+--------------------------------------+
| name               | String             | Object parameter name                |
+--------------------+--------------------+--------------------------------------+
| result             | String, enumerated | "SUCCESS", "FAIL\_UNKNOWN\_PARAM",   |
|                    |                    | "FAIL\_PARAM\_READONLY",             |
|                    |                    | "FAIL\_PARAM\_LOCKREQUIRED",         |
|                    |                    | "FAIL\_VALUE\_OUTOF\_RANGE",         |
|                    |                    | "FAIL\_VALUE\_TYPE\_ERROR"           |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "objId": "RxSigPath_5G:1",
            "globResult": "SUCCESS",
            "param": [
                {
                    "name": "dataLink",
                    "result": "SUCCESS"
                }
            ]
        }
    }

create-obj
~~~~~~~~~~

The Object Creation procedure allows the application to create and
initialize a new instance of the given object type on the RE.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:create-obj-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| objType    | String     | Object type          | RxSigPath\_5G        | Yes        |
+------------+------------+----------------------+----------------------+------------+
| name       | String     | Object parameter     | dataLink             | No         |
|            |            | name                 |                      |            |
+------------+------------+----------------------+----------------------+------------+
| value      | String     | Object parameter     | dataLink:1           | No         |
|            |            | value                |                      |            |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "create-obj-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "objType": "RxSigPath_5G",
                "param": [
                    {
                        "name": "dataLink",
                        "value": "dataLink:1"
                    }
                ]
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| objId              | String             | Object ID                            |
+--------------------+--------------------+--------------------------------------+
| globResult         | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJTYPE",            |
|                    |                    | "FAIL\_STATIC\_OBJTYPE",             |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_CHILD\_NOTALLOWED",           |
|                    |                    | "FAIL\_OUTOF\_RESOURCES"             |
|                    |                    | "FAIL\_PARAMETER\_FAIL",             |
|                    |                    | "FAIL\_NOSUCH\_RESOURCE"             |
+--------------------+--------------------+--------------------------------------+
| name               | String             | Object parameter name                |
+--------------------+--------------------+--------------------------------------+
| result             | String, enumerated | "SUCCESS", "FAIL\_UNKNOWN\_PARAM",   |
|                    |                    | "FAIL\_PARAM\_READONLY",             |
|                    |                    | "FAIL\_PARAM\_LOCKREQUIRED",         |
|                    |                    | "FAIL\_VALUE\_OUTOF\_RANGE",         |
|                    |                    | "FAIL\_VALUE\_TYPE\_ERROR"           |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "objId": "RxSigPath_5G:0",
            "globResult": "SUCCESS",
            "param": [
                {
                    "name": "dataLink",
                    "result": "SUCCESS"
                }
            ]
        }
    }

delete-obj
~~~~~~~~~~

The Object Deletion procedure allows the application to delete a given
object instance and recursively its entire child objects on the RE.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:delete-obj-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| objId      | String     | Object ID            | RxSigPath\_5G:1      | Yes        |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "delete-obj-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "obj-id": "RxSigPath_5G:0"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| result             | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_STATIC\_OBJTYPE",             |
|                    |                    | "FAIL\_LOCKREQUIRED"                 |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "result": "SUCCESS"
        }
    }

get-state
~~~~~~~~~

The Object State Reporting procedure allows the application to acquire
the current state (for the requested state type) of one or more objects
of the RE resource model, and additionally configure event-triggered
reporting of the detected state changes for all state types of the
indicated objects.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:get-state-nb

POST Input
^^^^^^^^^^

+--------------------+----------+--------------------+--------------------+----------+
| Field Name         | Type     | Description        | Example            | Required |
|                    |          |                    |                    | ?        |
+====================+==========+====================+====================+==========+
| nodeId             | String   | Inventory node     | ocp:MTI-101-200    | Yes      |
|                    |          | reference for OCP  |                    |          |
|                    |          | radio head         |                    |          |
+--------------------+----------+--------------------+--------------------+----------+
| objId              | String   | Object ID          | RxSigPath\_5G:1    | Yes      |
+--------------------+----------+--------------------+--------------------+----------+
| stateType          | String,  | Valid values:      | ALL                | Yes      |
|                    | enumerat | "AST", "FST",      |                    |          |
|                    | ed       | "ALL"              |                    |          |
+--------------------+----------+--------------------+--------------------+----------+
| eventDrivenReporti | Boolean  | Event-triggered    | true               | Yes      |
| ng                 |          | reporting of state |                    |          |
|                    |          | change             |                    |          |
+--------------------+----------+--------------------+--------------------+----------+

**Example.**

::

    {
        "get-state-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "objId": "antPort:0",
                "stateType": "ALL",
                "eventDrivenReporting": "true"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| id                 | String             | Object ID                            |
+--------------------+--------------------+--------------------------------------+
| type               | String, enumerated | State type. Valid values: "AST",     |
|                    |                    | "FST                                 |
+--------------------+--------------------+--------------------------------------+
| value              | String, enumerated | State value. Valid values: For state |
|                    |                    | type = "AST": "LOCKED", "UNLOCKED".  |
|                    |                    | For state type = "FST":              |
|                    |                    | "PRE\_OPERATIONAL", "OPERATIONAL",   |
|                    |                    | "DEGRADED", "FAILED",                |
|                    |                    | "NOT\_OPERATIONAL", "DISABLED"       |
+--------------------+--------------------+--------------------------------------+
| result             | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_UNKNOWN\_STATETYPE",          |
|                    |                    | "FAIL\_VALUE\_OUTOF\_RANGE"          |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "obj": [
                {
                    "id": "antPort:0",
                    "state": [
                        {
                            "type": "FST",
                            "value": "DISABLED"
                        },
                        {
                            "type": "AST",
                            "value": "LOCKED"
                        }
                    ]
                }
            ],
            "result": "SUCCESS"
        }
    }

modify-state
~~~~~~~~~~~~

The Object State Modification procedure allows the application to
trigger a change in the state of an object of the RE Resource Model.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:modify-state-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| objId      | String     | Object ID            | RxSigPath\_5G:1      | Yes        |
+------------+------------+----------------------+----------------------+------------+
| stateType  | String,    | Valid values: "AST", | AST                  | Yes        |
|            | enumerated | "FST", "ALL"         |                      |            |
+------------+------------+----------------------+----------------------+------------+
| stateValue | String,    | Valid values: For    | LOCKED               | Yes        |
|            | enumerated | state type = "AST":  |                      |            |
|            |            | "LOCKED",            |                      |            |
|            |            | "UNLOCKED". For      |                      |            |
|            |            | state type = "FST":  |                      |            |
|            |            | "PRE\_OPERATIONAL",  |                      |            |
|            |            | "OPERATIONAL",       |                      |            |
|            |            | "DEGRADED",          |                      |            |
|            |            | "FAILED",            |                      |            |
|            |            | "NOT\_OPERATIONAL",  |                      |            |
|            |            | "DISABLED"           |                      |            |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "modify-state-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "objId": "RxSigPath_5G:1",
                "stateType": "AST",
                "stateValue": "LOCKED"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| objId              | String             | Object ID                            |
+--------------------+--------------------+--------------------------------------+
| stateType          | String, enumerated | State type. Valid values: "AST",     |
|                    |                    | "FST                                 |
+--------------------+--------------------+--------------------------------------+
| stateValue         | String, enumerated | State value. Valid values: For state |
|                    |                    | type = "AST": "LOCKED", "UNLOCKED".  |
|                    |                    | For state type = "FST":              |
|                    |                    | "PRE\_OPERATIONAL", "OPERATIONAL",   |
|                    |                    | "DEGRADED", "FAILED",                |
|                    |                    | "NOT\_OPERATIONAL", "DISABLED"       |
+--------------------+--------------------+--------------------------------------+
| result             | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_UNKNOWN\_STATETYPE",          |
|                    |                    | "FAIL\_UNKNOWN\_STATEVALUE",         |
|                    |                    | "FAIL\_STATE\_READONLY",             |
|                    |                    | "FAIL\_RESOURCE\_UNAVAILABLE",       |
|                    |                    | "FAIL\_RESOURCE\_INUSE",             |
|                    |                    | "FAIL\_PARENT\_CHILD\_CONFLICT",     |
|                    |                    | "FAIL\_PRECONDITION\_NOTMET          |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "objId": "RxSigPath_5G:1",
            "stateType": "AST",
            "stateValue": "LOCKED",
            "result": "SUCCESS",
        }
    }

get-fault
~~~~~~~~~

The Fault Reporting procedure allows the application to acquire
information about all current active faults associated with a primary
object, as well as configure the RE to report when the fault status
changes for any of faults associated with the indicated primary object.

Default URL:
http://localhost:8181/restconf/operations/ocp-service:get-fault-nb

POST Input
^^^^^^^^^^

+------------+------------+----------------------+----------------------+------------+
| Field Name | Type       | Description          | Example              | Required?  |
+============+============+======================+======================+============+
| nodeId     | String     | Inventory node       | ocp:MTI-101-200      | Yes        |
|            |            | reference for OCP    |                      |            |
|            |            | radio head           |                      |            |
+------------+------------+----------------------+----------------------+------------+
| objId      | String     | Object ID            | RE:0                 | Yes        |
+------------+------------+----------------------+----------------------+------------+
| eventDrive | Boolean    | Event-triggered      | true                 | Yes        |
| nReporting |            | reporting of fault   |                      |            |
+------------+------------+----------------------+----------------------+------------+

**Example.**

::

    {
        "get-fault-nb": {
            "input": {
                "nodeId": "ocp:MTI-101-200",
                "objId": "RE:0",
                "eventDrivenReporting": "true"
            }
        }
    }

POST Output
^^^^^^^^^^^

+--------------------+--------------------+--------------------------------------+
| Field Name         | Type               | Description                          |
+====================+====================+======================================+
| result             | String, enumerated | Common default result codes +        |
|                    |                    | "FAIL\_UNKNOWN\_OBJECT",             |
|                    |                    | "FAIL\_VALUE\_OUTOF\_RANGE"          |
+--------------------+--------------------+--------------------------------------+
| id (obj)           | String             | Object ID                            |
+--------------------+--------------------+--------------------------------------+
| id (fault)         | String             | Fault ID                             |
+--------------------+--------------------+--------------------------------------+
| severity           | String             | Fault severity                       |
+--------------------+--------------------+--------------------------------------+
| timestamp          | dateTime           | Time stamp                           |
+--------------------+--------------------+--------------------------------------+
| descr              | String             | Text description                     |
+--------------------+--------------------+--------------------------------------+
| affectedObj        | String             | Affected object                      |
+--------------------+--------------------+--------------------------------------+

**Example.**

::

    {
        "output": {
            "result": "SUCCESS",
            "obj": [
                {
                    "id": "RE:0",
                    "fault": [
                        {
                            "id": "FAULT_OVERTEMP",
                            "severity": "DEGRADED",
                            "timestamp": "2012-02-12T16:35:00",
                            "descr": "PA temp too high; Pout reduced",
                            "affectedObj": [
                                "TxSigPath_EUTRA:0",
                                "TxSigPath_EUTRA:1"
                            ]
                        },
                        {
                            "id": "FAULT_VSWR_OUTOF_RANGE",
                            "severity": "WARNING",
                            "timestamp": "2012-02-12T16:01:05",
                        }
                    ]
                }
            ],
        }
    }

.. note::

    The northbound APIs described above wrap the southbound APIs to make
    them accessible to external applications via RESTCONF, as well as
    take care of synchronizing the RE resource model between radio heads
    and the controller’s datastore. See
    applications/ocp-service/src/main/yang/ocp-resourcemodel.yang for
    the yang representation of the RE resource model.

Java Interfaces (Southbound APIs)
---------------------------------

The southbound APIs provide concrete implementation of the following OCP
elementary functions: health-check, set-time, re-reset, get-param,
modify-param, create-obj, delete-obj, get-state, modify-state and
get-fault. Any OpenDaylight services/applications (of course, including
OCP service) wanting to speak OCP to radio heads will need to use them.

SalDeviceMgmtService
~~~~~~~~~~~~~~~~~~~~

Interface SalDeviceMgmtService defines three methods corresponding to
health-check, set-time and re-reset.

**SalDeviceMgmtService.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.device.mgmt.rev150811;

    public interface SalDeviceMgmtService
        extends
        RpcService
    {

        Future<RpcResult<HealthCheckOutput>> healthCheck(HealthCheckInput input);

        Future<RpcResult<SetTimeOutput>> setTime(SetTimeInput input);

        Future<RpcResult<ReResetOutput>> reReset(ReResetInput input);

    }

SalConfigMgmtService
~~~~~~~~~~~~~~~~~~~~

Interface SalConfigMgmtService defines two methods corresponding to
get-param and modify-param.

**SalConfigMgmtService.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.config.mgmt.rev150811;

    public interface SalConfigMgmtService
        extends
        RpcService
    {

        Future<RpcResult<GetParamOutput>> getParam(GetParamInput input);

        Future<RpcResult<ModifyParamOutput>> modifyParam(ModifyParamInput input);

    }

SalObjectLifecycleService
~~~~~~~~~~~~~~~~~~~~~~~~~

Interface SalObjectLifecycleService defines two methods corresponding to
create-obj and delete-obj.

**SalObjectLifecycleService.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.object.lifecycle.rev150811;

    public interface SalObjectLifecycleService
        extends
        RpcService
    {

        Future<RpcResult<CreateObjOutput>> createObj(CreateObjInput input);

        Future<RpcResult<DeleteObjOutput>> deleteObj(DeleteObjInput input);

    }

SalObjectStateMgmtService
~~~~~~~~~~~~~~~~~~~~~~~~~

Interface SalObjectStateMgmtService defines two methods corresponding to
get-state and modify-state.

**SalObjectStateMgmtService.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.object.state.mgmt.rev150811;

    public interface SalObjectStateMgmtService
        extends
        RpcService
    {

        Future<RpcResult<GetStateOutput>> getState(GetStateInput input);

        Future<RpcResult<ModifyStateOutput>> modifyState(ModifyStateInput input);

    }

SalFaultMgmtService
~~~~~~~~~~~~~~~~~~~

Interface SalFaultMgmtService defines only one method corresponding to
get-fault.

**SalFaultMgmtService.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.fault.mgmt.rev150811;

    public interface SalFaultMgmtService
        extends
        RpcService
    {

        Future<RpcResult<GetFaultOutput>> getFault(GetFaultInput input);

    }

Notifications
-------------

In addition to indication messages, the OCP southbound plugin will
translate specific events (e.g., connect, disconnect) coming up from the
OCP protocol library into MD-SAL Notification objects and then publish
them to the MD-SAL. Also, the OCP service will notify the completion of
certain operation via Notification as well.

SalDeviceMgmtListener
~~~~~~~~~~~~~~~~~~~~~

An onDeviceConnected Notification will be published to the MD-SAL as
soon as a radio head is connected to the controller, and when that radio
head is disconnected the OCP southbound plugin will publish an
onDeviceDisconnected Notification in response to the disconnect event
propagated from the OCP protocol library.

**SalDeviceMgmtListener.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.device.mgmt.rev150811;

    public interface SalDeviceMgmtListener
        extends
        NotificationListener
    {

        void onDeviceConnected(DeviceConnected notification);

        void onDeviceDisconnected(DeviceDisconnected notification);

    }

OcpServiceListener
~~~~~~~~~~~~~~~~~~

The OCP service will publish an onAlignmentCompleted Notification to the
MD-SAL once it has completed the OCP alignment procedure with the radio
head.

**OcpServiceListener.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.params.xml.ns.yang.ocp.applications.ocp.service.rev150811;

    public interface OcpServiceListener
        extends
        NotificationListener
    {

        void onAlignmentCompleted(AlignmentCompleted notification);

    }

SalObjectStateMgmtListener
~~~~~~~~~~~~~~~~~~~~~~~~~~

When receiving a state change indication message, the OCP southbound
plugin will propagate the indication message to upper layer
services/applications by publishing a corresponding onStateChangeInd
Notification to the MD-SAL.

**SalObjectStateMgmtListener.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.object.state.mgmt.rev150811;

    public interface SalObjectStateMgmtListener
        extends
        NotificationListener
    {

        void onStateChangeInd(StateChangeInd notification);

    }

SalFaultMgmtListener
~~~~~~~~~~~~~~~~~~~~

When receiving a fault indication message, the OCP southbound plugin
will propagate the indication message to upper layer
services/applications by publishing a corresponding onFaultInd
Notification to the MD-SAL.

**SalFaultMgmtListener.java.**

::

    package org.opendaylight.yang.gen.v1.urn.opendaylight.ocp.fault.mgmt.rev150811;

    public interface SalFaultMgmtListener
        extends
        NotificationListener
    {

        void onFaultInd(FaultInd notification);

    }
