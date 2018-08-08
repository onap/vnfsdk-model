.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017-2018 AT&T Intellectual Property, All rights reserved
.. Copyright 2017-2018 Huawei Technologies Co., Ltd.

===================================
Service: *VES Event Listener 7.0.1*
===================================

.. contents:: Table of Contents
Introduction
============

This document describes the RESTful interface for the VES Event
Listener. The VES acronym originally stood for Virtual-function Event
Streaming, but VES has been generalized to support network-function
event streaming, whether virtualized or not. The VES Event Listener is
capable of receiving any event sent in the VES Common Event Format. The
Common Event Format is expressed in JSON schema in section 4 of this
document. In the Common Event Format, an event consists of a required
Common Event Header block (i.e., object) accompanied by zero or more
event domain blocks.

It should be understood that events are well structured packages of
information, identified by an eventName, which are asynchronously
communicated to subscribers who are interested in the eventName. Events
can convey measurements, faults, syslogs, threshold crossing alerts and
other types of information. Events are simply a way of communicating
well-structured packages of information to one or more instances of an
Event Listener service.

This document describes a RESTful connectionless push event listener
that is capable of receiving single events or batches of events in the
Common Event Format. In future, additional documents may describe other
transports which make use of persistent TCP connections for high volumes
of streaming events.

Event Registration
------------------

All events must be compliant with the common event format, but specific
events identified by their eventNames, may require that certain fields,
which are optional in the common event format, be present when they are
published. For example, a specific eventName may require that specific
name-value pairs be present in the extensible structures provided within
the Common Event Format.

Events are registered using an extensible YAML format (defined in a
separate document), which specifies, for each eventName, the fields that
are required, what field values may be sent, and any special handling
that should be performed on those eventNames.

Naming Standards for eventName
------------------------------

To prevent naming collisions, eventNames sent as part of the
commonEventHeader, should conform to the following naming convention
designed to summarize the purpose and type of the event, and to ensure
the uniqueness of the eventName:

   {DomainAbbreviation}_{PublisherName}_{Description}

Each underscore-separated subfield above must start with a capital
letter and use camel-casing to separate words and acronyms. Acronyms
must capitalize only the first letter of the acronym. Spaces and
underscores should not appear within any subfield.

The DomainAbbreviation subfield derives from the ‘domain’ field in the
commonEventHeader, as specified below:

-  ‘Fault’ for the fault domain

-  ‘Heartbeat’ for the heartbeat domain

-  ‘Measurement’ for the measurements domain

-  ‘MobileFlow’ for the mobileFlow domain

-  ‘Other’ for the other domain

-  ‘PnfReg’ for the pnfRegistration domain

-  ‘SipSignaling’ for the sipSignaling domain

-  ‘StateChange’ for the stateChange domain

-  ‘Syslog’ for the syslog domain

-  ‘Tca’ for the thresholdCrossingAlert domain

-  ‘VoiceQuality’ for the voiceQuality domain

The PublisherName subfield describes the vendor product or application
publishing the event. This subfield conforms to the following
conventions:

-  Vendor products are specified as:

{productName}-{vendorName}

   For example: Visbc-Metaswitch or Vdbe-Juniper, where a hyphen is used
   to separate the productName and vendorName subfields. Note that the
   productName and vendorName subfields must not include hyphens
   themselves.

   Organizing the information in this way will cause an alphabetical
   listing of eventNames to sort similar network functions together,
   rather than to sort them by vendor.

   The productName subfield may describe a NF or a NFC. Where NFC names
   may be reused across different NF’s, they should be specified as:

{NfName}:{NfcName}

   where a colon is used to separate the NfName and NfcName subfields.
   Note that the NfName and NfcName subfields must not include colons
   themselves.

   The ProductName may also describe other types of vendor modules or
   components such as a VM, application or hostname. As with NFs and
   NFCs, parent:child relationships may be communicated using colon as a
   subfield delimiter.

-  Service providers who adopt the VES Common Event Format for internal
   use, may provide PublisherName without the vendorName subfield. They
   would typically identify an application, system, service or
   microservice publishing the event (e.g., ‘Policy’, ‘So’,
   ‘MobileCallRecording’ or ‘Dkat’). As with NFs and NFCs, parent:child
   relationships may be communicated using colon as a subfield delimiter
   (e.g., ApplicationName:ApplicationComponent).

The final subfield of the eventName name should describe, in a compact
camel case format the specific information being conveyed by the event.
In some cases, this final subfield may not be required (e.g., in the
case of certain heartbeats).

Examples of eventNames following the naming standards are provided
below:

-  Tca_Vdbe-Ericsson_CpuThresholdExceeded

-  Heartbeat_Visbc:Mmc-Metaswitch

-  Syslog_Vdbe-Ericsson

-  Fault_MobileCallRecording_PilotNumberPoolExhaustion

-  Other_So:WanBonding_InstantiationPart1Complete

EventId Use Cases Examples
--------------------------

[Author: Alok Gupta]:

eventId Examples:

Example 1: assumes a unique key for each domain consisting of domain
followed by an integer domainnnnnnn e.g. fault000001, heartbeat000001,
mfvs000005

Example 2: assumes an integer key for all events nnnnnnnnn: 000000001,
00000002, 000000003

Rules:

1. All domains except Fault: each time a subsequent event is sent the
   integer part of eventId will increment by 1. Repeat of eventId
   assumes duplicate event. Sequence number is set to 0 for all domains
   except fault.

2. eventId construction for Fault Events:

   a. Most likely scenario

      i.   The sourceName on each Fault event is the vNFC Name or VM
           hostname.

      ii.  The eventId on Fault events is the same every time a given
           fault is raised (onset), re-raised at fixed time interval,
           until it is cleared. Once the fault is cleared, a new eventId
           is used.

      iii. The startEpochMicrosec value for the Fault event is the
           timestamp for when that event is generated until a clear is
           sent.

      iv.  lastEpochMicrosec indicates the current event time.

      v.   The sequence number for each Fault event is set to 1 when the
           event is first raised, and increments each time the same
           Fault event is raised, until a clear is sent.

           |image0|

   b. Alternative scenario: for vNF when fault event status is not
      maintained.

      vi.   The sourceName on each Fault event is the vNFC Name or VM
            hostname.

      vii.  The eventId on Fault events is the same every time a given
            fault is raised or cleared, even if it is re-raised after it
            had previously cleared.  So, for example, if EMS loses
            contact with a particular device then a Fault event might be
            sent for a raise, re-raise (because EMS has re-tried and
            still can’t contact the device), clear (because EMS has
            re-tried and it can contact the device) and then raise again
            (because EMS has lost contact with the device again).  The
            same eventId is used for all 4 of those Fault events.

      viii. The startEpochMicrosec value for each Fault event is the
            timestamp for when that event is generated, not when the
            fault first occurred.  So all 4 of the Fault events in the
            previous bullet point would have a different timestamp.

      ix.   lastEpochMicrosec indicates the current event time.

      x.    The sequence number for each Fault event is currently set to
            0 on a raise and 1 on a clear.  We could change that so that
            each Fault event is given a new monotonically increasing
            sequence number whether it is a raise or a clear if that is
            helpful (which is reset to 0 if the VM restarts) but they
            won’t be consecutive.

|image1|

Measurement Expansion Fields
----------------------------

When expansion fields are used, the goal is to avoid custom development
by the service provider collecting the fields, since custom development
adds obvious cost, delay and resource overhead. In the domain of
measurements, it is expected that a high percentage (perhaps as high as
90 percent) of use cases for extensible fields can be satisfied by using
the additionalMeasurements arrayOfNamedHashMap data structure in
combination with a YAML registration file (provided at design time). The
YAML registration file conveys meta-information about the processing of
additionalMeasurements. For more information, please see the VES Event
Registration specification and in particular the aggregationRole, castTo
and isHomogeneous keywords.

Syslogs
-------

Syslog’s can be classified as either Control or Session/Traffic. They
differ by message content and expected volume: 

-  Control logs are generally free-form human-readable text used for
   reporting errors or warnings supporting the operation and
   troubleshooting of NFs.  The volume of these logs is typically less
   than 2k per day.

-  Session logs use common structured fields to report normal NF
   processing such as DNS lookups or firewall rules processed.  The
   volume of these logs is typically greater than 1k per hour (and
   sometimes as high as 10k per second).

VES supports both classes of syslog, however VES is only recommended for
control logs or for lower volume session logs, less than 60k per hour.
High volume session logging should use a file-based transport solution.

Support for Protocols Other Than HTTPS
--------------------------------------

This API specification describes an HTTPS RESTful interface using the
JSON content-type.

Alternative API specifications may be provided in future using Google
Protobuf, websockets, or Apache Avro.

Versioning
----------

Three types of version numbers supported by this specification:

-  The API specification itself is versioned. Going forward, the major
   number of the specification version will be incremented whenever any
   change could break an existing client (e.g., a field name is deleted
   or changed). All other changes to the spec (e.g., a field name is
   added or text changes are made to the specification itself) will
   increment only the minor number or patch number. Note that the major
   number appears in REST resource URLs as v# (where ‘#’ is the major
   number). Minor and patch numbers are communicated in HTTP headers.
   For more information, see the API Versioning writeup in section 6.1.

-  The JSON schema is versioned. Going forward, the major number of the
   JSON schema will be incremented whenever any change could break an
   existing client (e.g., a field name is deleted or changed). All other
   changes to the schema (e.g., a field name is added or text changes
   are made to the field descriptions) will increment only the minor
   number or patch number.

-  The field blocks are versioned. Field blocks include the
   commonEventHeader and the domain blocks (e.g., the faultFields
   block). Going forward, the major number of each field block will be
   incremented whenever any change to that block could break an existing
   client (e.g., a field name is deleted or changed). All other changes
   to that block (e.g., a field name is added or text changes are made
   to the field descriptions) will increment only the minor number.

Field Block Versions
~~~~~~~~~~~~~~~~~~~~

A summary of the latest field block versions as of this version of the
API spec is provided below:

-  commonEventHeader version:4.0.1

-  commonEventHeader vesEventListenerVersion: 7.0.1

-  faultFieldsVersion:4.0

-  heartbeatFieldsVersion: 3.0

-  measurementFieldsVersion: 4.0

-  mobileFlowFieldsVersion: 4.0

-  notificationFieldsVersion: 2.0

-  otherFieldsVersion: 3.0

-  pnfRegistrationFieldsVersion: 2.0

-  sigSignalingFieldsVersion: 3.0

-  stateChangeFieldsVersion: 4.0

-  syslogFieldsVersion: 4.0

-  thresholdCrossingFieldsVersion: 4.0

-  voiceQualityFieldsVersion: 4.0

Security
========

Event sources must identify themselves to the VES Event Listener.

In the future, support for 2-way SSL certificate authentication (aka
mutual SSL) will be provided but for now, event source credentials are
passed using HTTP `Basic
Authentication <http://tools.ietf.org/html/rfc2617>`__.

Credentials must not be passed on the query string. Credentials must be
sent in an Authorization header as follows:

1. The username and password are formed into one string as
   “username:password”

2. The resulting string is Base64 encoded to produce the encoded
   credential.

3. The encoded credential is communicated in the header after the string
   “Authorization: Basic “

Because the credentials are merely encoded but not encrypted, HTTPS
(rather than HTTP) should be used. HTTPS will also encrypt and protect
event contents. TLS 1.2 or higher must be used.

Examples are provided below.

Sample Request and Response
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sample Request
^^^^^^^^^^^^^^

+---------------------------------------------------------------+
| POST /eventListener/v7 HTTP/1.1                               |
|                                                               |
| Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==             |
|                                                               |
| content-type: application/json                                |
|                                                               |
| | content-length: 12345                                       |
| | {                                                           |
|                                                               |
| "event": {                                                    |
|                                                               |
| "commonEventHeader": {                                        |
|                                                               |
| "version": "4.0.1",                                           |
|                                                               |
| "vesEventListenerVersion": "7.0.1",                           |
|                                                               |
| "domain": "heartbeat",                                        |
|                                                               |
| "eventName": "Heartbeat_vIsbcMmc",                            |
|                                                               |
| "eventId": "heartbeat0000249",                                |
|                                                               |
| "sequence": 0,                                                |
|                                                               |
| "priority": "Normal",                                         |
|                                                               |
|  "reportingEntityId": "cc305d54-75b4-431b-adb2-eb6b9e541234", |
|                                                               |
| "reportingEntityName": "ibcx0001vm002oam001",                 |
|                                                               |
| "sourceId": "de305d54-75b4-431b-adb2-eb6b9e546014",           |
|                                                               |
| "sourceName": "ibcx0001vm002ssc001",                          |
|                                                               |
| "nfVendorName": "Ericsson",                                   |
|                                                               |
| "nfNamingCode": "ibcx",                                       |
|                                                               |
| "nfcNamingCode": "ssc",                                       |
|                                                               |
| "startEpochMicrosec": 1413378172000000,                       |
|                                                               |
| "lastEpochMicrosec": 1413378172000000,                        |
|                                                               |
| "timeZoneOffset": "UTC-05:30"                                 |
|                                                               |
| }                                                             |
|                                                               |
| }                                                             |
|                                                               |
| }                                                             |
+---------------------------------------------------------------+

Sample Success Response
^^^^^^^^^^^^^^^^^^^^^^^

+------------------------+
| HTTPS/1.1 202 Accepted |
|                        |
| X-MinorVersion: 0      |
|                        |
| X-PatchVersion: 1      |
|                        |
| X-LatestVersion: 7.0.1 |
+------------------------+

Resource Structure
==================

REST resources are defined with respect to a ServerRoot:

ServerRoot = /{optionalRoutingtPath}

The resource structure is provided below:

|image2|

Figure 1 – REST Resource Structure

The {Domain} or FQDN above is typically provisioned into each
eventsource when it is instantiated. The {Port} above is typically 8443.

Common Event Format
===================

A JSON schema describing the Common Event Format is provided below and
is reproduced in the tables that follow.

Note on optional fields:

   If the event publisher collects a field that is identified as
   optional in the data structures below, then the event publisher
   *must* send that field.

Note on extensible fields:

   VES contains various extensible structures (e.g., hashMap) that
   enable event publishers to send information that has not been
   explicitly defined in VES data structures.

-  Event publishers *must not* send information through extensible
   structures where VES has explicitly defined fields for that
   information. For example, event publishers *must not* send
   information like cpuIdle, through an extensible structure, because
   VES has explicitly defined a cpuUsage.cpuIdle field for the
   communication of that information.

-  Keys sent through extensible fields must use camel casing to separate
   words and acronyms; only the first letter of each acronym shall be
   capitalized.

Common Event Datatypes
----------------------

.. _common-event-datatypes-1:

Common Event Datatypes
~~~~~~~~~~~~~~~~~~~~~~

Datatype: arrayOfJsonObject
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The arrayOfJsonObject datatype provides an array of json objects, each
of which is described by name, schema and other meta-information. It
consists of the following fields:

+-------------------+----------------+-----------+---------------------+
| Field             | Type           | Required? | Description         |
+===================+================+===========+=====================+
| arrayOfJsonObject | jsonObject [ ] | Yes       | Array of jsonObject |
+-------------------+----------------+-----------+---------------------+

Datatype: arrayOfNamedHashMap
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The arrayOfNamedHashMap datatype provides an array of hashMaps, each of
which is associated with a descriptive name. It consists of the
following fields:

+---------------------+------------------+-----------+-----------------------+
| Field               | Type             | Required? | Description           |
+=====================+==================+===========+=======================+
| arrayOfNamedHashMap | namedHashMap [ ] | Yes       | Array of namedHashMap |
+---------------------+------------------+-----------+-----------------------+

Datatype: event
^^^^^^^^^^^^^^^

The event datatype consists of the following fields which constitute the
‘root level’ of the common event format:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| commonEventHead | commonEventHead | Yes             | Fields common   |
| er              | er              |                 | to all events   |
+-----------------+-----------------+-----------------+-----------------+
| faultFields     | faultFields     | No              | Fields specific |
|                 |                 |                 | to fault events |
+-----------------+-----------------+-----------------+-----------------+
| heartbeatFields | heartbeatFields | No              | Fields specific |
|                 |                 |                 | to heartbeat    |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| measurementFiel | measurementFiel | No              | Fields specific |
| ds              | ds              |                 | to measurement  |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| mobileFlowField | mobileFlowField | No              | Fields specific |
| s               | s               |                 | to mobility     |
|                 |                 |                 | flow events     |
+-----------------+-----------------+-----------------+-----------------+
| notificationFie | notificationFie | No              | Fields specific |
| lds             | lds             |                 | to notification |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| otherFields     | otherFields     | No              | Fields specific |
|                 |                 |                 | to other types  |
|                 |                 |                 | of events       |
+-----------------+-----------------+-----------------+-----------------+
| pnfRegistration | pnfRegistration | No              | Fields specific |
| Fields          | Fields          |                 | to              |
|                 |                 |                 | pnfRegistration |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| sipSignalingFie | sipSignalingFie | No              | Fields specific |
| lds             | lds             |                 | to sipSignaling |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| stateChangeFiel | stateChangeFiel | No              | Fields specific |
| ds              | ds              |                 | to state change |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| syslogFields    | syslogFields    | No              | Fields specific |
|                 |                 |                 | to syslog       |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| thresholdCrossi | thresholdCrossi | No              | Fields specific |
| ngAlertFields   | ngAlertFields   |                 | to threshold    |
|                 |                 |                 | crossing alert  |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+
| voiceQualityFie | voiceQualityFie | No              | Fields specific |
| lds             | lds             |                 | to voiceQuality |
|                 |                 |                 | events          |
+-----------------+-----------------+-----------------+-----------------+

Datatype: eventList
^^^^^^^^^^^^^^^^^^^

The eventList datatype consists of the following fields:

+-----------+-----------+-----------+-----------------+
| Field     | Type      | Required? | Description     |
+===========+===========+===========+=================+
| eventList | event [ ] | Yes       | Array of events |
+-----------+-----------+-----------+-----------------+

Datatype: hashMap
^^^^^^^^^^^^^^^^^

The hashMap datatype is an ‘associative array’, which is an unordered
collection of key-value pairs of the form “key”: “value”, where each key
and value are strings. Keys must use camel casing to separate words and
acronyms; only the first letter of each acronym shall be capitalized.

Datatype: jsonObject
^^^^^^^^^^^^^^^^^^^^

The jsonObject datatype provides a json object schema, name and other
meta-information along with one or more object instances that conform to
the schema:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| objectInstances | JsonObjectInsta | Yes             | Contains one or |
|                 | nce             |                 | more instances  |
|                 | [ ]             |                 | of the json     |
|                 |                 |                 | object          |
+-----------------+-----------------+-----------------+-----------------+
| objectName      | string          | Yes             | Name of the     |
|                 |                 |                 | json object     |
+-----------------+-----------------+-----------------+-----------------+
| objectSchema    | string          | No              | json schema for |
|                 |                 |                 | the object      |
+-----------------+-----------------+-----------------+-----------------+
| objectSchemaUrl | string          | No              | URL to the json |
|                 |                 |                 | schema for the  |
|                 |                 |                 | object          |
+-----------------+-----------------+-----------------+-----------------+
| nfSubscribedObj | string          | No              | Name of the     |
| ectName         |                 |                 | object          |
|                 |                 |                 | associated with |
|                 |                 |                 | the             |
|                 |                 |                 | nfSubscriptionI |
|                 |                 |                 | d               |
+-----------------+-----------------+-----------------+-----------------+
| nfSubscriptionI | string          | No              | Identifies an   |
| d               |                 |                 | openConfig      |
|                 |                 |                 | telemetry       |
|                 |                 |                 | subscription on |
|                 |                 |                 | a network       |
|                 |                 |                 | function, which |
|                 |                 |                 | configures the  |
|                 |                 |                 | network         |
|                 |                 |                 | function to     |
|                 |                 |                 | send complex    |
|                 |                 |                 | object data     |
|                 |                 |                 | associated with |
|                 |                 |                 | the jsonObject  |
+-----------------+-----------------+-----------------+-----------------+

Datatype: jsonObjectInstance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The jsonObjectInstance datatype provides meta-information about an
instance of a jsonObject along with the actual object instance:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| jsonObject      | jsonObject      | No              | Optional        |
|                 |                 |                 | recursive       |
|                 |                 |                 | specification   |
|                 |                 |                 | of jsonObject   |
+-----------------+-----------------+-----------------+-----------------+
| objectInstance  | object          | No              | Contains an     |
|                 |                 |                 | instance        |
|                 |                 |                 | conforming to   |
|                 |                 |                 | the jsonObject  |
|                 |                 |                 | schema          |
+-----------------+-----------------+-----------------+-----------------+
| objectInstanceE | number          | No              | the unix time,  |
| pochMicrosec    |                 |                 | aka epoch time, |
|                 |                 |                 | associated with |
|                 |                 |                 | this            |
|                 |                 |                 | objectInstance- |
|                 |                 |                 | -as             |
|                 |                 |                 | microseconds    |
|                 |                 |                 | elapsed since 1 |
|                 |                 |                 | Jan 1970 not    |
|                 |                 |                 | including leap  |
|                 |                 |                 | seconds         |
+-----------------+-----------------+-----------------+-----------------+
| objectKeys      | key [ ]         | No              | An ordered set  |
|                 |                 |                 | of keys that    |
|                 |                 |                 | identifies this |
|                 |                 |                 | particular      |
|                 |                 |                 | instance of     |
|                 |                 |                 | jsonObject      |
|                 |                 |                 | (e.g., that     |
|                 |                 |                 | places it in a  |
|                 |                 |                 | hierarchy)      |
+-----------------+-----------------+-----------------+-----------------+

Datatype: key
^^^^^^^^^^^^^

The key datatype is a tuple which provides the name of a key along with
its value and relative order; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| keyName         | string          | Yes             | Name of the key |
+-----------------+-----------------+-----------------+-----------------+
| keyOrder        | Integer         | No              | Relative        |
|                 |                 |                 | sequence or     |
|                 |                 |                 | order of the    |
|                 |                 |                 | key (with       |
|                 |                 |                 | respect to      |
|                 |                 |                 | other keys)     |
+-----------------+-----------------+-----------------+-----------------+
| keyValue        | string          | No              | Value of the    |
|                 |                 |                 | key             |
+-----------------+-----------------+-----------------+-----------------+

Datatype: namedHashMap
^^^^^^^^^^^^^^^^^^^^^^

The namedHashMap datatype is a hashMap which is associated with and
described by a name; it consists of the following fields:

+---------+---------+-----------+------------------------------------------------+
| Field   | Type    | Required? | Description                                    |
+=========+=========+===========+================================================+
| name    | string  | Yes       | Name associated with or describing the hashmap |
+---------+---------+-----------+------------------------------------------------+
| hashMap | hashMap | Yes       | One or more key:value pairs                    |
+---------+---------+-----------+------------------------------------------------+

Datatype: requestError
^^^^^^^^^^^^^^^^^^^^^^

The requestError datatype defines the standard request error data
structure:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| messageId       | string          | Yes             | Unique message  |
|                 |                 |                 | identifier of   |
|                 |                 |                 | the format      |
|                 |                 |                 | ‘ABCnnnn’ where |
|                 |                 |                 | ‘ABC’ is either |
|                 |                 |                 | ‘SVC’ for       |
|                 |                 |                 | Service         |
|                 |                 |                 | Exceptions or   |
|                 |                 |                 | ‘POL’ for       |
|                 |                 |                 | Policy          |
|                 |                 |                 | Exception.      |
|                 |                 |                 | Exception       |
|                 |                 |                 | numbers may be  |
|                 |                 |                 | in the range of |
|                 |                 |                 | 0001 to 9999    |
|                 |                 |                 | where 0001 to   |
|                 |                 |                 | 2999 are        |
|                 |                 |                 | defined by OMA  |
|                 |                 |                 | (see section    |
|                 |                 |                 | 5.1) and        |
|                 |                 |                 | 3000-9999 are   |
|                 |                 |                 | available and   |
|                 |                 |                 | undefined.      |
+-----------------+-----------------+-----------------+-----------------+
| text            | string          | Yes             | Message text,   |
|                 |                 |                 | with            |
|                 |                 |                 | replacement     |
|                 |                 |                 | variables       |
|                 |                 |                 | marked with %n, |
|                 |                 |                 | where n is an   |
|                 |                 |                 | index into the  |
|                 |                 |                 | list of         |
|                 |                 |                 | <variables>     |
|                 |                 |                 | elements,       |
|                 |                 |                 | starting at 1   |
+-----------------+-----------------+-----------------+-----------------+
| url             | string          | No              | Hyperlink to a  |
|                 |                 |                 | detailed error  |
|                 |                 |                 | resource e.g.,  |
|                 |                 |                 | an HTML page    |
|                 |                 |                 | for browser     |
|                 |                 |                 | user agents     |
+-----------------+-----------------+-----------------+-----------------+
| variables       | string          | No              | List of zero or |
|                 |                 |                 | more strings    |
|                 |                 |                 | that represent  |
|                 |                 |                 | the contents of |
|                 |                 |                 | the variables   |
|                 |                 |                 | used by the     |
|                 |                 |                 | message text    |
+-----------------+-----------------+-----------------+-----------------+

Datatype: vendorNfNameFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The vendorNfNameFields provides vendor, nf and nfModule identifying
information:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| vendorName      | string          | Yes             | Network         |
|                 |                 |                 | function vendor |
|                 |                 |                 | name            |
+-----------------+-----------------+-----------------+-----------------+
| nfModuleName    | string          | No              | Name of the     |
|                 |                 |                 | nfModule        |
|                 |                 |                 | generating the  |
|                 |                 |                 | event           |
+-----------------+-----------------+-----------------+-----------------+
| nfName          | string          | No              | Name of the     |
|                 |                 |                 | network         |
|                 |                 |                 | function        |
|                 |                 |                 | generating the  |
|                 |                 |                 | event           |
+-----------------+-----------------+-----------------+-----------------+

‘Common Event Header’ Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: commonEventHeader
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The commonEventHeader datatype consists of the following fields common
to all events:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| domain          | string          | Yes             | Event domain    |
|                 |                 |                 | enumeration:    |
|                 |                 |                 | ‘fault’,        |
|                 |                 |                 | ‘heartbeat’,    |
|                 |                 |                 | ‘measurement’,  |
|                 |                 |                 | ‘mobileFlow’,   |
|                 |                 |                 | ‘notification’, |
|                 |                 |                 | ‘other’,        |
|                 |                 |                 | ‘pnfRegistratio |
|                 |                 |                 | n’,             |
|                 |                 |                 | ‘sipSignaling’, |
|                 |                 |                 | ‘stateChange’,  |
|                 |                 |                 | ‘syslog’,       |
|                 |                 |                 | ‘thresholdCross |
|                 |                 |                 | ingAlert’,      |
|                 |                 |                 | ‘voiceQuality’  |
+-----------------+-----------------+-----------------+-----------------+
| eventId         | string          | Yes             | Event key that  |
|                 |                 |                 | is unique to    |
|                 |                 |                 | the event       |
|                 |                 |                 | source.         |
|                 |                 |                 | The key must be |
|                 |                 |                 |  unique within  |
|                 |                 |                 | notification    |
|                 |                 |                 | life cycle      |
|                 |                 |                 | similar to      |
|                 |                 |                 | EventID from    |
|                 |                 |                 | 3GPP. It could  |
|                 |                 |                 | be a sequential |
|                 |                 |                 | number, or a    |
|                 |                 |                 | composite key   |
|                 |                 |                 | formed from the |
|                 |                 |                 | event fields,   |
|                 |                 |                 | such as         |
|                 |                 |                 | domain_sequence |
|                 |                 |                 | .               |
|                 |                 |                 | The eventId     |
|                 |                 |                 | should not      |
|                 |                 |                 | include         |
|                 |                 |                 | whitespace. For |
|                 |                 |                 | fault events,   |
|                 |                 |                 | eventId is the  |
|                 |                 |                 | eventId of the  |
|                 |                 |                 | initial alarm;  |
|                 |                 |                 | if the same     |
|                 |                 |                 | alarm is raised |
|                 |                 |                 | again for       |
|                 |                 |                 | changed,        |
|                 |                 |                 | acknowledged or |
|                 |                 |                 | cleared cases,  |
|                 |                 |                 | eventId must be |
|                 |                 |                 | the same as the |
|                 |                 |                 | initial alarm   |
|                 |                 |                 | (along with the |
|                 |                 |                 | same            |
|                 |                 |                 | startEpochMicro |
|                 |                 |                 | sec             |
|                 |                 |                 | but with a      |
|                 |                 |                 | different       |
|                 |                 |                 | sequence        |
|                 |                 |                 | number). Note:  |
|                 |                 |                 | see section 1.3 |
|                 |                 |                 | for eventId use |
|                 |                 |                 | case examples.  |
+-----------------+-----------------+-----------------+-----------------+
| eventName       | string          | Yes             | Unique event    |
|                 |                 |                 | name (see       |
|                 |                 |                 | section 1.2 for |
|                 |                 |                 | more            |
|                 |                 |                 | information)    |
+-----------------+-----------------+-----------------+-----------------+
| eventType       | string          | No              | For example:    |
|                 |                 |                 | ‘applicationNf’ |
|                 |                 |                 | ,               |
|                 |                 |                 | ‘guestOS’,      |
|                 |                 |                 | ‘hostOS’,       |
|                 |                 |                 | ‘platform’      |
+-----------------+-----------------+-----------------+-----------------+
| internalHeader  | internalHeader  | No              | Fields (not     |
| Fields          | Fields          |                 | supplied by     |
|                 |                 |                 | event sources)  |
|                 |                 |                 | that the VES    |
|                 |                 |                 | Event Listener  |
|                 |                 |                 | service can use |
|                 |                 |                 | to enrich the   |
|                 |                 |                 | event if needed |
|                 |                 |                 | for efficient   |
|                 |                 |                 | internal        |
|                 |                 |                 | processing.     |
|                 |                 |                 | This is an      |
|                 |                 |                 | empty object    |
|                 |                 |                 | which is        |
|                 |                 |                 | intended to be  |
|                 |                 |                 | defined         |
|                 |                 |                 | separately by   |
|                 |                 |                 | each service    |
|                 |                 |                 | provider (e.g., |
|                 |                 |                 | AT&T)           |
|                 |                 |                 | implementing    |
|                 |                 |                 | the VES Event   |
|                 |                 |                 | Listener.       |
+-----------------+-----------------+-----------------+-----------------+
| lastEpochMicros | number          | Yes             | the latest unix |
| ec              |                 |                 | time aka epoch  |
|                 |                 |                 | time associated |
|                 |                 |                 | with the event  |
|                 |                 |                 | from any        |
|                 |                 |                 | component--as   |
|                 |                 |                 | microseconds    |
|                 |                 |                 | elapsed since 1 |
|                 |                 |                 | Jan 1970 not    |
|                 |                 |                 | including leap  |
|                 |                 |                 | seconds         |
+-----------------+-----------------+-----------------+-----------------+
| nfcNamingCode   | string          | No              | Network         |
|                 |                 |                 | function        |
|                 |                 |                 | component type: |
|                 |                 |                 | 3 characters    |
|                 |                 |                 | (aligned with   |
|                 |                 |                 | vfc naming      |
|                 |                 |                 | standards)      |
+-----------------+-----------------+-----------------+-----------------+
| nfNamingCode    | string          | No              | Network         |
|                 |                 |                 | function type:  |
|                 |                 |                 | 4 characters    |
|                 |                 |                 | (aligned with   |
|                 |                 |                 | vnf and pnf     |
|                 |                 |                 | naming          |
|                 |                 |                 | standards)      |
+-----------------+-----------------+-----------------+-----------------+
| nfVendorName    | string          | No              | Network         |
|                 |                 |                 | function vendor |
|                 |                 |                 | name            |
+-----------------+-----------------+-----------------+-----------------+
| priority        | string          | Yes             | Processing      |
|                 |                 |                 | priority        |
|                 |                 |                 | enumeration:    |
|                 |                 |                 | ‘High’,         |
|                 |                 |                 | ‘Medium’,       |
|                 |                 |                 | ‘Normal’, ‘Low’ |
+-----------------+-----------------+-----------------+-----------------+
| reportingEntity | string          | No              | UUID            |
| Id              |                 |                 | identifying the |
|                 |                 |                 | entity          |
|                 |                 |                 | reporting the   |
|                 |                 |                 | event or        |
|                 |                 |                 | detecting a     |
|                 |                 |                 | problem in      |
|                 |                 |                 | another vnf/vm  |
|                 |                 |                 | or pnf which is |
|                 |                 |                 | experiencing    |
|                 |                 |                 | the problem.    |
|                 |                 |                 | (Note: the AT&T |
|                 |                 |                 | internal        |
|                 |                 |                 | enrichment      |
|                 |                 |                 | process shall   |
|                 |                 |                 | ensure that     |
|                 |                 |                 | this field is   |
|                 |                 |                 | populated). The |
|                 |                 |                 | reportingEntity |
|                 |                 |                 | Id              |
|                 |                 |                 | is an id for    |
|                 |                 |                 | the             |
|                 |                 |                 | reportingEntity |
|                 |                 |                 | Name.           |
|                 |                 |                 | See             |
|                 |                 |                 | ‘reportingEntit |
|                 |                 |                 | yName’          |
|                 |                 |                 | for more        |
|                 |                 |                 | information.    |
+-----------------+-----------------+-----------------+-----------------+
| reportingEntity | string          | Yes             | Name of the     |
| Name            |                 |                 | entity          |
|                 |                 |                 | reporting the   |
|                 |                 |                 | event or        |
|                 |                 |                 | detecting a     |
|                 |                 |                 | problem in      |
|                 |                 |                 | another vnf/vm  |
|                 |                 |                 | or pnf which is |
|                 |                 |                 | experiencing    |
|                 |                 |                 | the problem.    |
|                 |                 |                 | May be the same |
|                 |                 |                 | as the          |
|                 |                 |                 | sourceName. For |
|                 |                 |                 | synthetic       |
|                 |                 |                 | events          |
|                 |                 |                 | generated by    |
|                 |                 |                 | DCAE, it is the |
|                 |                 |                 | name of the app |
|                 |                 |                 | generating the  |
|                 |                 |                 | event.          |
+-----------------+-----------------+-----------------+-----------------+
| sequence        | integer         | Yes             | Ordering of     |
|                 |                 |                 | events          |
|                 |                 |                 | communicated by |
|                 |                 |                 | an event source |
|                 |                 |                 | instance (or 0  |
|                 |                 |                 | if not needed)  |
+-----------------+-----------------+-----------------+-----------------+
| sourceId        | string          | No              | UUID            |
|                 |                 |                 | identifying the |
|                 |                 |                 | entity          |
|                 |                 |                 | experiencing    |
|                 |                 |                 | the event       |
|                 |                 |                 | issue, which    |
|                 |                 |                 | may be detected |
|                 |                 |                 | and reported by |
|                 |                 |                 | a separate      |
|                 |                 |                 | reporting       |
|                 |                 |                 | entity (note:   |
|                 |                 |                 | the AT&T        |
|                 |                 |                 | internal        |
|                 |                 |                 | enrichment      |
|                 |                 |                 | process shall   |
|                 |                 |                 | ensure that     |
|                 |                 |                 | this field is   |
|                 |                 |                 | populated). The |
|                 |                 |                 | sourceId is an  |
|                 |                 |                 | id for the      |
|                 |                 |                 | sourceName. See |
|                 |                 |                 | ‘sourceName’    |
|                 |                 |                 | for more        |
|                 |                 |                 | information.    |
+-----------------+-----------------+-----------------+-----------------+
| sourceName      | string          | Yes             | Name of the     |
|                 |                 |                 | entity          |
|                 |                 |                 | experiencing    |
|                 |                 |                 | the event       |
|                 |                 |                 | issue, which    |
|                 |                 |                 | may be detected |
|                 |                 |                 | and reported by |
|                 |                 |                 | a separate      |
|                 |                 |                 | reporting       |
|                 |                 |                 | entity. The     |
|                 |                 |                 | sourceName      |
|                 |                 |                 | identifies the  |
|                 |                 |                 | device for      |
|                 |                 |                 | which data is   |
|                 |                 |                 | collected. A    |
|                 |                 |                 | valid           |
|                 |                 |                 | sourceName must |
|                 |                 |                 | be inventoried  |
|                 |                 |                 | in A&AI. If     |
|                 |                 |                 | sourceName is a |
|                 |                 |                 | xNFC or VM,     |
|                 |                 |                 | then the event  |
|                 |                 |                 | must be         |
|                 |                 |                 | reporting data  |
|                 |                 |                 | for that        |
|                 |                 |                 | particular xNFC |
|                 |                 |                 | or VM. If the   |
|                 |                 |                 | sourceName is a |
|                 |                 |                 | xNF, comprised  |
|                 |                 |                 | of multiple     |
|                 |                 |                 | xNFCs, the data |
|                 |                 |                 | must be         |
|                 |                 |                 | reported/aggreg |
|                 |                 |                 | ated            |
|                 |                 |                 | at the xNF      |
|                 |                 |                 | leveI.  Data    |
|                 |                 |                 | for individual  |
|                 |                 |                 | xNFC must not   |
|                 |                 |                 | be included in  |
|                 |                 |                 | the xNF         |
|                 |                 |                 | sourceName      |
|                 |                 |                 | event.          |
+-----------------+-----------------+-----------------+-----------------+
| startEpochMicro | number          | Yes             | the earliest    |
| sec             |                 |                 | unix time aka   |
|                 |                 |                 | epoch time      |
|                 |                 |                 | associated with |
|                 |                 |                 | the event from  |
|                 |                 |                 | any             |
|                 |                 |                 | component--as   |
|                 |                 |                 | microseconds    |
|                 |                 |                 | elapsed since 1 |
|                 |                 |                 | Jan 1970 not    |
|                 |                 |                 | including leap  |
|                 |                 |                 | seconds. For    |
|                 |                 |                 | measurements    |
|                 |                 |                 | and heartbeats, |
|                 |                 |                 | where events    |
|                 |                 |                 | are collected   |
|                 |                 |                 | over predefined |
|                 |                 |                 | intervals,      |
|                 |                 |                 | startEpochMicro |
|                 |                 |                 | sec             |
|                 |                 |                 | shall be        |
|                 |                 |                 | rounded to the  |
|                 |                 |                 | nearest         |
|                 |                 |                 | interval        |
|                 |                 |                 | boundary (e.g., |
|                 |                 |                 | the epoch       |
|                 |                 |                 | equivalent of   |
|                 |                 |                 | 3:00PM, 3:10PM, |
|                 |                 |                 | 3:20PM, etc…).  |
|                 |                 |                 | For fault       |
|                 |                 |                 | events,         |
|                 |                 |                 | startEpochMicro |
|                 |                 |                 | sec             |
|                 |                 |                 | is the          |
|                 |                 |                 | timestamp of    |
|                 |                 |                 | the initial     |
|                 |                 |                 | alarm; if the   |
|                 |                 |                 | same alarm is   |
|                 |                 |                 | raised again    |
|                 |                 |                 | for changed,    |
|                 |                 |                 | acknowledged or |
|                 |                 |                 | cleared cases,  |
|                 |                 |                 | startEpoch      |
|                 |                 |                 | Microsec must   |
|                 |                 |                 | be the same as  |
|                 |                 |                 | the initial     |
|                 |                 |                 | alarm (along    |
|                 |                 |                 | with the same   |
|                 |                 |                 | eventId and an  |
|                 |                 |                 | incremental     |
|                 |                 |                 | sequence        |
|                 |                 |                 | number). For    |
|                 |                 |                 | devices with no |
|                 |                 |                 | timing source   |
|                 |                 |                 | (clock), the    |
|                 |                 |                 | default value   |
|                 |                 |                 | will be 0 and   |
|                 |                 |                 | the VES         |
|                 |                 |                 | collector will  |
|                 |                 |                 | replace it with |
|                 |                 |                 | Collector time  |
|                 |                 |                 | stamp (when the |
|                 |                 |                 | event is        |
|                 |                 |                 | received)       |
+-----------------+-----------------+-----------------+-----------------+
| timeZoneOffset  | string          | No              | Offset to GMT   |
|                 |                 |                 | to indicate     |
|                 |                 |                 | local time zone |
|                 |                 |                 | for device      |
|                 |                 |                 | formatted as    |
|                 |                 |                 | ‘UTC+/-hh.mm’;  |
|                 |                 |                 | see             |
|                 |                 |                 | https://en.wiki |
|                 |                 |                 | pedia.org/wiki/ |
|                 |                 |                 | List_of_time_zo |
|                 |                 |                 | ne_abbreviation |
|                 |                 |                 | s               |
|                 |                 |                 | for UTC offset  |
|                 |                 |                 | examples        |
+-----------------+-----------------+-----------------+-----------------+
| version         | string          | Yes             | Version of the  |
|                 |                 |                 | event header as |
|                 |                 |                 | “#.#” where #   |
|                 |                 |                 | is a digit; see |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| vesEventListene | string          | Yes             | Version of the  |
| rVersion        |                 |                 | ves event       |
|                 |                 |                 | listener api    |
|                 |                 |                 | spec that this  |
|                 |                 |                 | event is        |
|                 |                 |                 | compliant with  |
|                 |                 |                 | (as “#” or      |
|                 |                 |                 | “#.#” or        |
|                 |                 |                 | “#.#.#” where # |
|                 |                 |                 | is a digit; see |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use). |
+-----------------+-----------------+-----------------+-----------------+

Datatype: internalHeaderFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The internalHeaderFields datatype is an undefined object which can
contain arbitrarily complex JSON structures. It is intended to be
defined separately by each service provider (e.g., AT&T) implementing
the VES Event Listener. The fields in internalHeaderFields are not
provided by any event source but instead are added by the VES Event
Listener service itself as part of an event enrichment process necessary
for efficient internal processing of events received by the VES Event
Listener.

Technology Independent Datatypes
--------------------------------

‘Fault’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: faultFields
^^^^^^^^^^^^^^^^^^^^^

The faultFields datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| alarmAdditional | hashMap         | No              | Additional      |
| Information     |                 |                 | alarm           |
|                 |                 |                 | information.    |
|                 |                 |                 |                 |
|                 |                 |                 | -  Note1: for   |
|                 |                 |                 |    SNMP mapping |
|                 |                 |                 |    to VES, for  |
|                 |                 |                 |    hash key use |
|                 |                 |                 |    OID of       |
|                 |                 |                 |    varbind, for |
|                 |                 |                 |    value use    |
|                 |                 |                 |    incoming     |
|                 |                 |                 |    data for     |
|                 |                 |                 |    that         |
|                 |                 |                 |    varbind).    |
|                 |                 |                 |                 |
|                 |                 |                 | -  Note2: Alarm |
|                 |                 |                 |    ID for 3GPP  |
|                 |                 |                 |    should be    |
|                 |                 |                 |    included (if |
|                 |                 |                 |    applicable)  |
|                 |                 |                 |    in           |
|                 |                 |                 |    alarmAdditon |
|                 |                 |                 | alInformation   |
|                 |                 |                 |    as           |
|                 |                 |                 |    ‘alarmId’:’a |
|                 |                 |                 | larmIdValue’.   |
|                 |                 |                 |                 |
|                 |                 |                 | Could contain   |
|                 |                 |                 | managed object  |
|                 |                 |                 | instance as     |
|                 |                 |                 | separate        |
|                 |                 |                 | key:value;      |
|                 |                 |                 | could add       |
|                 |                 |                 | probable cause  |
|                 |                 |                 | as separate     |
|                 |                 |                 | key:value.      |
+-----------------+-----------------+-----------------+-----------------+
| alarmCondition  | string          | Yes             | Short name of   |
|                 |                 |                 | the alarm       |
|                 |                 |                 | condition/probl |
|                 |                 |                 | em,             |
|                 |                 |                 | such as a trap  |
|                 |                 |                 | name. Should    |
|                 |                 |                 | not have white  |
|                 |                 |                 | space (e.g.,    |
|                 |                 |                 | tpLgCgiNotInCon |
|                 |                 |                 | fig,            |
|                 |                 |                 | BfdSessionDown, |
|                 |                 |                 | linkDown, etc…) |
+-----------------+-----------------+-----------------+-----------------+
| alarmInterfaceA | string          | No              | Card, port,     |
|                 |                 |                 | channel or      |
|                 |                 |                 | interface name  |
|                 |                 |                 | of the device   |
|                 |                 |                 | generating the  |
|                 |                 |                 | alarm. This     |
|                 |                 |                 | could reflect   |
|                 |                 |                 | managed object. |
+-----------------+-----------------+-----------------+-----------------+
| eventCategory   | string          | No              | Event category, |
|                 |                 |                 | for example:    |
|                 |                 |                 | ‘license’,      |
|                 |                 |                 | ‘link’,         |
|                 |                 |                 | ‘routing’,      |
|                 |                 |                 | ‘security’,     |
|                 |                 |                 | ‘signaling’     |
+-----------------+-----------------+-----------------+-----------------+
| eventSeverity   | string          | Yes             | Event severity  |
|                 |                 |                 | enumeration:    |
|                 |                 |                 | ‘CRITICAL’,     |
|                 |                 |                 | ‘MAJOR’,        |
|                 |                 |                 | ‘MINOR’,        |
|                 |                 |                 | ‘WARNING’,      |
|                 |                 |                 | ‘NORMAL’.       |
|                 |                 |                 | NORMAL is used  |
|                 |                 |                 | to represent    |
|                 |                 |                 | clear.          |
+-----------------+-----------------+-----------------+-----------------+
| eventSourceType | string          | Yes             | Examples:       |
|                 |                 |                 | ‘card’, ‘host’, |
|                 |                 |                 | ‘other’,        |
|                 |                 |                 | ‘port’,         |
|                 |                 |                 | ‘portThreshold’ |
|                 |                 |                 | ,               |
|                 |                 |                 | ‘router’,       |
|                 |                 |                 | ‘slotThreshold’ |
|                 |                 |                 | ,               |
|                 |                 |                 | ‘switch’,       |
|                 |                 |                 | ‘virtualMachine |
|                 |                 |                 | ’,              |
|                 |                 |                 | ‘virtualNetwork |
|                 |                 |                 | Function’.      |
|                 |                 |                 | This could be   |
|                 |                 |                 | managed object  |
|                 |                 |                 | class.          |
+-----------------+-----------------+-----------------+-----------------+
| faultFieldsVers | string          | Yes             | Version of the  |
| ion             |                 |                 | faultFields     |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| specificProblem | string          | Yes             | Description of  |
|                 |                 |                 | the alarm or    |
|                 |                 |                 | problem (e.g.,  |
|                 |                 |                 | ‘eNodeB 155197  |
|                 |                 |                 | in PLMN 310-410 |
|                 |                 |                 | with eNodeB     |
|                 |                 |                 | name KYL05197   |
|                 |                 |                 | is lost’). 3GPP |
|                 |                 |                 | probable cause  |
|                 |                 |                 | would be        |
|                 |                 |                 | included in     |
|                 |                 |                 | this field.     |
+-----------------+-----------------+-----------------+-----------------+
| vfStatus        | string          | Yes             | Virtual         |
|                 |                 |                 | function status |
|                 |                 |                 | enumeration:    |
|                 |                 |                 | ‘Active’,       |
|                 |                 |                 | ‘Idle’,         |
|                 |                 |                 | ‘Preparing to   |
|                 |                 |                 | terminate’,     |
|                 |                 |                 | ‘Ready to       |
|                 |                 |                 | terminate’,     |
|                 |                 |                 | ‘Requesting     |
|                 |                 |                 | Termination’    |
+-----------------+-----------------+-----------------+-----------------+

‘Heartbeat’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: heartbeatFields
^^^^^^^^^^^^^^^^^^^^^^^^^

The heartbeatFields datatype is an optional field block for fields
specific to heartbeat events; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | expansion       |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| heartbeatFields | string          | Yes             | Version of the  |
| Version         |                 |                 | heartbeatFields |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| heartbeatInterv | Integer         | Yes             | Current         |
| al              |                 |                 | heartbeatInterv |
|                 |                 |                 | al              |
|                 |                 |                 | in seconds      |
+-----------------+-----------------+-----------------+-----------------+

 ‘Measurements’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note: NFs are required to report exactly one Measurement event per
period per sourceName.

Datatype: codecsInUse
^^^^^^^^^^^^^^^^^^^^^

The codecsInUse datatype consists of the following fields describing the
number of times an identified codec was used over the
measurementInterval:

+----------------+---------+-----------+------------------------------+
| Field          | Type    | Required? | Description                  |
+================+=========+===========+==============================+
| codecIdentifer | string  | Yes       | Description of the codec     |
+----------------+---------+-----------+------------------------------+
| numberInUse    | integer | Yes       | Number of such codecs in use |
+----------------+---------+-----------+------------------------------+

Datatype: cpuUsage
^^^^^^^^^^^^^^^^^^

The cpuUsage datatype defines the usage of an identifier CPU and
consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| cpuCapacityCont | number          | No              | The amount of   |
| ention          |                 |                 | time the CPU    |
|                 |                 |                 | cannot run due  |
|                 |                 |                 | to contention,  |
|                 |                 |                 | in milliseconds |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| cpuDemandAvg    | number          | No              | The total CPU   |
|                 |                 |                 | time that the   |
|                 |                 |                 | NF/NFC/VM could |
|                 |                 |                 | use if there    |
|                 |                 |                 | was no          |
|                 |                 |                 | contention, in  |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| cpuDemandMhz    | number          | No              | CPU demand in   |
|                 |                 |                 | MHz             |
+-----------------+-----------------+-----------------+-----------------+
| cpuDemandPct    | number          | No              | CPU demand as a |
|                 |                 |                 | percentage of   |
|                 |                 |                 | the provisioned |
|                 |                 |                 | capacity        |
+-----------------+-----------------+-----------------+-----------------+
| cpuIdentifier   | string          | Yes             | CPU Identifier  |
+-----------------+-----------------+-----------------+-----------------+
| cpuIdle         | number          | No              | Percentage of   |
|                 |                 |                 | CPU time spent  |
|                 |                 |                 | in the idle     |
|                 |                 |                 | task            |
+-----------------+-----------------+-----------------+-----------------+
| cpuLatencyAvg   | number          | No              | Percentage of   |
|                 |                 |                 | time the VM is  |
|                 |                 |                 | unable to run   |
|                 |                 |                 | because it is   |
|                 |                 |                 | contending for  |
|                 |                 |                 | access to the   |
|                 |                 |                 | physical CPUs   |
+-----------------+-----------------+-----------------+-----------------+
| cpuOverheadAvg  | number          | No              | The overhead    |
|                 |                 |                 | demand above    |
|                 |                 |                 | available       |
|                 |                 |                 | allocations and |
|                 |                 |                 | reservations,   |
|                 |                 |                 | in milliseconds |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| cpuSwapWaitTime | number          | No              | Swap wait time, |
|                 |                 |                 | in milliseconds |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageInterru | number          | No              | Percentage of   |
| pt              |                 |                 | time spent      |
|                 |                 |                 | servicing       |
|                 |                 |                 | interrupts      |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageNice    | number          | No              | Percentage of   |
|                 |                 |                 | time spent      |
|                 |                 |                 | running user    |
|                 |                 |                 | space processes |
|                 |                 |                 | that have been  |
|                 |                 |                 | niced           |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageSoftIrq | number          | No              | Percentage of   |
|                 |                 |                 | time spent      |
|                 |                 |                 | handling soft   |
|                 |                 |                 | irq interrupts  |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageSteal   | number          | No              | Percentage of   |
|                 |                 |                 | time spent in   |
|                 |                 |                 | involuntary     |
|                 |                 |                 | wait which is   |
|                 |                 |                 | neither user,   |
|                 |                 |                 | system or idle  |
|                 |                 |                 | time and is     |
|                 |                 |                 | effectively     |
|                 |                 |                 | time that went  |
|                 |                 |                 | missing         |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageSystem  | number          | No              | Percentage of   |
|                 |                 |                 | time spent on   |
|                 |                 |                 | system tasks    |
|                 |                 |                 | running the     |
|                 |                 |                 | kernel          |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageUser    | number          | No              | Percentage of   |
|                 |                 |                 | time spent      |
|                 |                 |                 | running         |
|                 |                 |                 | un-niced user   |
|                 |                 |                 | space processes |
+-----------------+-----------------+-----------------+-----------------+
| cpuWait         | number          | No              | Percentage of   |
|                 |                 |                 | CPU time spent  |
|                 |                 |                 | waiting for I/O |
|                 |                 |                 | operations to   |
|                 |                 |                 | complete        |
+-----------------+-----------------+-----------------+-----------------+
| percentUsage    | number          | Yes             | Aggregate cpu   |
|                 |                 |                 | usage of the    |
|                 |                 |                 | virtual machine |
|                 |                 |                 | on which the    |
|                 |                 |                 | xNFC reporting  |
|                 |                 |                 | the event is    |
|                 |                 |                 | running         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: diskUsage
^^^^^^^^^^^^^^^^^^^

The diskUsage datatype defines the usage of a disk and consists of the
following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| diskBusResets   | number          | No              | Number of bus   |
|                 |                 |                 | resets over the |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| diskCommandsAbo | number          | No              | Number of disk  |
| rted            |                 |                 | commands        |
|                 |                 |                 | aborted over    |
|                 |                 |                 | the             |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| diskCommandsAvg | number          | No              | Average number  |
|                 |                 |                 | of commands per |
|                 |                 |                 | second over the |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| diskFlushReques | number          | No              | Total flush     |
| ts              |                 |                 | requests of the |
|                 |                 |                 | disk cache over |
|                 |                 |                 | the             |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| diskFlushTime   | number          | No              | Milliseconds    |
|                 |                 |                 | spent on disk   |
|                 |                 |                 | cache flushing  |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| diskIdentifier  | string          | Yes             | Disk Identifier |
+-----------------+-----------------+-----------------+-----------------+
| diskIoTimeAvg   | number          | No              | Milliseconds    |
|                 |                 |                 | spent doing     |
|                 |                 |                 | input/output    |
|                 |                 |                 | operations over |
|                 |                 |                 | 1 sec; treat    |
|                 |                 |                 | this metric as  |
|                 |                 |                 | a device load   |
|                 |                 |                 | percentage      |
|                 |                 |                 | where 1000ms    |
|                 |                 |                 | matches 100%    |
|                 |                 |                 | load; provide   |
|                 |                 |                 | the average     |
|                 |                 |                 | over the        |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskIoTimeLast  | number          | No              | Milliseconds    |
|                 |                 |                 | spent doing     |
|                 |                 |                 | input/output    |
|                 |                 |                 | operations over |
|                 |                 |                 | 1 sec; treat    |
|                 |                 |                 | this metric as  |
|                 |                 |                 | a device load   |
|                 |                 |                 | percentage      |
|                 |                 |                 | where 1000ms    |
|                 |                 |                 | matches 100%    |
|                 |                 |                 | load; provide   |
|                 |                 |                 | the last value  |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskIoTimeMax   | number          | No              | Milliseconds    |
|                 |                 |                 | spent doing     |
|                 |                 |                 | input/output    |
|                 |                 |                 | operations over |
|                 |                 |                 | 1 sec; treat    |
|                 |                 |                 | this metric as  |
|                 |                 |                 | a device load   |
|                 |                 |                 | percentage      |
|                 |                 |                 | where 1000ms    |
|                 |                 |                 | matches 100%    |
|                 |                 |                 | load; provide   |
|                 |                 |                 | the maximum     |
|                 |                 |                 | value           |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskIoTimeMin   | number          | No              | Milliseconds    |
|                 |                 |                 | spent doing     |
|                 |                 |                 | input/output    |
|                 |                 |                 | operations over |
|                 |                 |                 | 1 sec; treat    |
|                 |                 |                 | this metric as  |
|                 |                 |                 | a device load   |
|                 |                 |                 | percentage      |
|                 |                 |                 | where 1000ms    |
|                 |                 |                 | matches 100%    |
|                 |                 |                 | load; provide   |
|                 |                 |                 | the minimum     |
|                 |                 |                 | value           |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedReadA | number          | No              | Number of       |
| vg              |                 |                 | logical read    |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | read            |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical reads   |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedReadL | number          | No              | Number of       |
| ast             |                 |                 | logical read    |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | read            |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical reads   |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | last value      |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedReadM | number          | No              | Number of       |
| ax              |                 |                 | logical read    |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | read            |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical reads   |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum value   |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedReadM | number          | No              | Number of       |
| in              |                 |                 | logical read    |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | read            |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical reads   |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum value   |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedWrite | number          | No              | Number of       |
| Avg             |                 |                 | logical write   |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | write           |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical writes  |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedWrite | number          | No              | Number of       |
| Last            |                 |                 | logical write   |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | write           |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical writes  |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | last value      |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedWrite | number          | No              | Number of       |
| Max             |                 |                 | logical write   |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | write           |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical writes  |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum value   |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskMergedWrite | number          | No              | Number of       |
| Min             |                 |                 | logical write   |
|                 |                 |                 | operations that |
|                 |                 |                 | were merged     |
|                 |                 |                 | into physical   |
|                 |                 |                 | write           |
|                 |                 |                 | operations,     |
|                 |                 |                 | e.g., two       |
|                 |                 |                 | logical writes  |
|                 |                 |                 | were served by  |
|                 |                 |                 | one physical    |
|                 |                 |                 | disk access;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum value   |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsRead  | number          | No              | Number of       |
| Avg             |                 |                 | octets per      |
|                 |                 |                 | second read     |
|                 |                 |                 | from a disk or  |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsRead  | number          | No              | Number of       |
|                 |                 |                 | octets per      |
| Last            |                 |                 | second read     |
|                 |                 |                 | from a disk or  |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | last            |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsRead  | number          | No              | Number of       |
| Max             |                 |                 | octets per      |
|                 |                 |                 | second read     |
|                 |                 |                 | from a disk or  |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsRead  | number          | No              | Number of       |
| Min             |                 |                 | octets per      |
|                 |                 |                 | second read     |
|                 |                 |                 | from a disk or  |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsWrite | number          | No              | Number of       |
| Avg             |                 |                 | octets per      |
|                 |                 |                 | second written  |
|                 |                 |                 | to a disk or    |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsWrite | number          | No              | Number of       |
| Last            |                 |                 | octets per      |
|                 |                 |                 | second written  |
|                 |                 |                 | to a disk or    |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | last            |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsWrite | number          | No              | Number of       |
| Max             |                 |                 | octets per      |
|                 |                 |                 | second written  |
|                 |                 |                 | to a disk or    |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOctetsWrite | number          | No              | Number of       |
| Min             |                 |                 | octets per      |
|                 |                 |                 | second written  |
|                 |                 |                 | to a disk or    |
|                 |                 |                 | partition;      |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsReadAvg  | number          | No              | Number of read  |
|                 |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsReadLast | number          | No              | Number of read  |
|                 |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | last            |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsReadMax  | number          | No              | Number of read  |
|                 |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsReadMin  | number          | No              | Number of read  |
|                 |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsWriteAvg | number          | No              | Number of write |
|                 |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsWriteLas | number          | No              | Number of write |
| t               |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | last            |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsWrite    | number          | No              | Number of write |
| Max             |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskOpsWriteMin | number          | No              | Number of write |
|                 |                 |                 | operations per  |
|                 |                 |                 | second issued   |
|                 |                 |                 | to the disk;    |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskPendingOper | number          | No              | Queue size of   |
| ationsAvg       |                 |                 | pending I/O     |
|                 |                 |                 | operations per  |
|                 |                 |                 | second; provide |
|                 |                 |                 | the average     |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskPendingOper | number          | No              | Queue size of   |
| ationsLast      |                 |                 | pending I/O     |
|                 |                 |                 | operations per  |
|                 |                 |                 | second; provide |
|                 |                 |                 | the last        |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskPendingOper | number          | No              | Queue size of   |
| ationsMax       |                 |                 | pending I/O     |
|                 |                 |                 | operations per  |
|                 |                 |                 | second; provide |
|                 |                 |                 | the maximum     |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskPendingOper | number          | No              | Queue size of   |
| ationsMin       |                 |                 | pending I/O     |
|                 |                 |                 | operations per  |
|                 |                 |                 | second; provide |
|                 |                 |                 | the minimum     |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskReadCommand | number          | No              | Average number  |
| sAvg            |                 |                 | of read         |
|                 |                 |                 | commands issued |
|                 |                 |                 | per second to   |
|                 |                 |                 | the disk over   |
|                 |                 |                 | the             |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| diskTime        | number          | No              | Nanoseconds     |
|                 |                 |                 | spent on disk   |
|                 |                 |                 | cache           |
|                 |                 |                 | reads/writes    |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeReadAvg | number          | No              | Milliseconds a  |
|                 |                 |                 | read operation  |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeRead    | number          | No              | Milliseconds a  |
| Last            |                 |                 | read operation  |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | last            |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeRead    | number          | No              | Milliseconds a  |
| Max             |                 |                 | read operation  |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeRead    | number          | No              | Milliseconds a  |
| Min             |                 |                 | read operation  |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeWrite   | number          | No              | Milliseconds a  |
| Avg             |                 |                 | write operation |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | average         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeWrite   | number          | No              | Milliseconds a  |
| Last            |                 |                 | write operation |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | last            |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeWrite   | number          | No              | Milliseconds a  |
| Max             |                 |                 | write operation |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | maximum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTimeWrite   | number          | No              | Milliseconds a  |
| Min             |                 |                 | write operation |
|                 |                 |                 | took to         |
|                 |                 |                 | complete;       |
|                 |                 |                 | provide the     |
|                 |                 |                 | minimum         |
|                 |                 |                 | measurement     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTotalReadLa | number          | No              | Average read    |
| tencyAvg        |                 |                 | time from the   |
|                 |                 |                 | perspective of  |
|                 |                 |                 | a Guest OS: sum |
|                 |                 |                 | of the Kernel   |
|                 |                 |                 | Read Latency    |
|                 |                 |                 | and Physical    |
|                 |                 |                 | Device Read     |
|                 |                 |                 | Latency in      |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | over the        |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskTotalWriteL | number          | No              | Average write   |
| atencyAvg       |                 |                 | time from the   |
|                 |                 |                 | perspective of  |
|                 |                 |                 | a Guest OS: sum |
|                 |                 |                 | of the Kernel   |
|                 |                 |                 | Write Latency   |
|                 |                 |                 | and Physical    |
|                 |                 |                 | Device Write    |
|                 |                 |                 | Latency in      |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | over the        |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| diskWeightedIoT | number          | No              | Measure in ms   |
| imeAvg          |                 |                 | over 1 sec of   |
|                 |                 |                 | both I/O        |
|                 |                 |                 | completion time |
|                 |                 |                 | and the backlog |
|                 |                 |                 | that may be     |
|                 |                 |                 | accumulating.   |
|                 |                 |                 | Value is the    |
|                 |                 |                 | average within  |
|                 |                 |                 | the collection  |
|                 |                 |                 | interval.       |
+-----------------+-----------------+-----------------+-----------------+
| diskWeightedIoT | number          | No              | Measure in ms   |
| imeLast         |                 |                 | over 1 sec of   |
|                 |                 |                 | both I/O        |
|                 |                 |                 | completion time |
|                 |                 |                 | and the backlog |
|                 |                 |                 | that may be     |
|                 |                 |                 | accumulating.   |
|                 |                 |                 | Value is the    |
|                 |                 |                 | last within the |
|                 |                 |                 | collection      |
|                 |                 |                 | interval.       |
+-----------------+-----------------+-----------------+-----------------+
| diskWeightedIoT | number          | No              | Measure in ms   |
| imeMax          |                 |                 | over 1 sec of   |
|                 |                 |                 | both I/O        |
|                 |                 |                 | completion time |
|                 |                 |                 | and the backlog |
|                 |                 |                 | that may be     |
|                 |                 |                 | accumulating.   |
|                 |                 |                 | Value is the    |
|                 |                 |                 | maximum within  |
|                 |                 |                 | the collection  |
|                 |                 |                 | interval.       |
+-----------------+-----------------+-----------------+-----------------+
| diskWeightedIoT | number          | No              | Measure in ms   |
| imeMin          |                 |                 | over 1 sec of   |
|                 |                 |                 | both I/O        |
|                 |                 |                 | completion time |
|                 |                 |                 | and the backlog |
|                 |                 |                 | that may be     |
|                 |                 |                 | accumulating.   |
|                 |                 |                 | Value is the    |
|                 |                 |                 | minimum within  |
|                 |                 |                 | the collection  |
|                 |                 |                 | interval.       |
+-----------------+-----------------+-----------------+-----------------+
| diskWriteComman | number          | No              | Average number  |
| dsAvg           |                 |                 | of write        |
|                 |                 |                 | commands issued |
|                 |                 |                 | per second to   |
|                 |                 |                 | the disk over   |
|                 |                 |                 | the             |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+

Datatype: filesystemUsage
^^^^^^^^^^^^^^^^^^^^^^^^^

The filesystemUsage datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| filesystemName  | string          | Yes             | File system     |
|                 |                 |                 | name            |
+-----------------+-----------------+-----------------+-----------------+
| blockConfigured | number          | Yes             | Configured      |
|                 |                 |                 | block storage   |
|                 |                 |                 | capacity in GB  |
+-----------------+-----------------+-----------------+-----------------+
| blockIops       | number          | Yes             | Block storage   |
|                 |                 |                 | input-output    |
|                 |                 |                 | operations per  |
|                 |                 |                 | second          |
+-----------------+-----------------+-----------------+-----------------+
| blockUsed       | number          | Yes             | Used block      |
|                 |                 |                 | storage         |
|                 |                 |                 | capacity in GB  |
+-----------------+-----------------+-----------------+-----------------+
| ephemeralConfig | number          | Yes             | Configured      |
| ured            |                 |                 | ephemeral       |
|                 |                 |                 | storage         |
|                 |                 |                 | capacity in GB  |
+-----------------+-----------------+-----------------+-----------------+
| ephemeralIops   | number          | Yes             | Ephemeral       |
|                 |                 |                 | storage         |
|                 |                 |                 | input-output    |
|                 |                 |                 | operations per  |
|                 |                 |                 | second          |
+-----------------+-----------------+-----------------+-----------------+
| ephemeralUsed   | number          | Yes             | Used ephemeral  |
|                 |                 |                 | storage         |
|                 |                 |                 | capacity in GB  |
+-----------------+-----------------+-----------------+-----------------+

Datatype: hugePages
^^^^^^^^^^^^^^^^^^^

The hugePages datatype provides metrics on system hugePages; it consists
of the following fields:

+---------------------+--------+-----------+-------------------------------------+
| Field               | Type   | Required? | Description                         |
+=====================+========+===========+=====================================+
| bytesFree           | number | No        | Number of free hugePages in bytes   |
+---------------------+--------+-----------+-------------------------------------+
| bytesUsed           | number | No        | Number of used hugePages in bytes   |
+---------------------+--------+-----------+-------------------------------------+
| hugePagesIdentifier | string | Yes       | HugePages identifier                |
+---------------------+--------+-----------+-------------------------------------+
| percentFree         | number | No        | Number of free hugePages in percent |
+---------------------+--------+-----------+-------------------------------------+
| percentUsed         | number | No        | Number of used hugePages in percent |
+---------------------+--------+-----------+-------------------------------------+
| vmPageNumberFree    | number | No        | Number of free vmPages in numbers   |
+---------------------+--------+-----------+-------------------------------------+
| vmPageNumberUsed    | number | No        | Number of used vmPages in numbers   |
+---------------------+--------+-----------+-------------------------------------+

Datatype: ipmi (Intelligent Platform Management Interface)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ipmi datatype provides intelligent platform management interface
metrics; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| exitAirTemperat | number          | No              | System fan exit |
| ure             |                 |                 | air flow        |
|                 |                 |                 | temperature in  |
|                 |                 |                 | Celsius         |
+-----------------+-----------------+-----------------+-----------------+
| frontPanelTempe | number          | No              | Front panel     |
| rature          |                 |                 | temp in Celsius |
+-----------------+-----------------+-----------------+-----------------+
| ioModuleTempera | number          | No              | Io module temp  |
| ture            |                 |                 | in Celsius      |
+-----------------+-----------------+-----------------+-----------------+
| ipmiBaseboardTe | ipmiBaseboard   | No              | Array of        |
| mperatureArray  | Temperature [ ] |                 | ipmiBaseboard   |
|                 |                 |                 | Temperature     |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| ipmiBaseboardVo | ipmiBaseboard   | No              | Array of        |
| ltageRegulator  | VoltageRegulato |                 | ipmiBaseboard   |
| Array           | r               |                 | VoltageRegulato |
|                 | [ ]             |                 | r               |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| ipmiBatteryArra | ipmiBattery [ ] | No              | Array of        |
| y               |                 |                 | ipmiBattery     |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| ipmiFanArray    | ipmiFan [ ]     | No              | Array of        |
|                 |                 |                 | ipmiFan objects |
+-----------------+-----------------+-----------------+-----------------+
| ipmiGlobalAggre | ipmiGlobalAggre | No              | ipmi global     |
| gateTemperature | gate            |                 | aggregate       |
|                 |                 |                 | temperature     |
| MarginArray     | TemperatureMarg |                 | margin          |
|                 | in              |                 |                 |
|                 | [ ]             |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| ipmiHsbpArray   | ipmiHsbp [ ]    | No              | Array of        |
|                 |                 |                 | ipmiHsbp        |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| ipmiNicArray    | ipmiNic [ ]     | No              | Array of        |
|                 |                 |                 | ipmiNic objects |
+-----------------+-----------------+-----------------+-----------------+
| ipmiPowerSupply | ipmiPowerSupply | No              | Array of        |
| Array           | [ ]             |                 | ipmiPowerSupply |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| ipmiProcessorAr | ipmiProcessor [ | No              | Array of        |
| ray             | ]               |                 | ipmiProcessor   |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| systemAirflow   | number          | No              | Airflow in      |
|                 |                 |                 | cubic feet per  |
|                 |                 |                 | minute (cfm)    |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiBaseboardTemperature
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ipmiBaseboardTemperature datatype consists of the following fields
which describe ipmi baseboard temperature metrics:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| baseboardTemper | number          | No              | Baseboard       |
| ature           |                 |                 | temperature in  |
|                 |                 |                 | celsius         |
+-----------------+-----------------+-----------------+-----------------+
| baseboardTemper | string          | Yes             | Identifier for  |
| ature           |                 |                 | the location    |
| Identifier      |                 |                 | where the       |
|                 |                 |                 | temperature is  |
|                 |                 |                 | taken           |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiBaseboardVoltageRegulator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ipmiBaseboardVoltageRegulator datatype consists of the following
fields which describe ipmi baseboard voltage regulator metrics:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| baseboardVoltag | string          | Yes             | Identifier for  |
| e               |                 |                 | the baseboard   |
| RegulatorIdenti |                 |                 | voltage         |
| fier            |                 |                 | regulator       |
+-----------------+-----------------+-----------------+-----------------+
| voltageRegulato | number          | No              | Voltage         |
| r               |                 |                 | regulator       |
| Temperature     |                 |                 | temperature in  |
|                 |                 |                 | celsius         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiBattery
^^^^^^^^^^^^^^^^^^^^^

The ipmiBattery datatype consists of the following fields which describe
ipmi battery metrics:

+---------------------+--------+-----------+----------------------------+
| Field               | Type   | Required? | Description                |
+=====================+========+===========+============================+
| batteryIdentifier   | string | Yes       | Identifier for the battery |
+---------------------+--------+-----------+----------------------------+
| batteryType         | string | No        | Type of battery            |
+---------------------+--------+-----------+----------------------------+
| batteryVoltageLevel | number | No        | Battery voltage level      |
+---------------------+--------+-----------+----------------------------+

Datatype: ipmiFan
^^^^^^^^^^^^^^^^^

The ipmiFan datatype consists of the following fields which describe
ipmi fan metrics:

+---------------+--------+-----------+-------------------------------------------+
| Field         | Type   | Required? | Description                               |
+===============+========+===========+===========================================+
| fanIdentifier | string | Yes       | Identifier for the fan                    |
+---------------+--------+-----------+-------------------------------------------+
| fanSpeed      | number | No        | Fan speed in revolutions per minute (rpm) |
+---------------+--------+-----------+-------------------------------------------+

Datatype: ipmiGlobalAggregateTemperatureMargin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ipmiGlobalAggregateTemperatureMargin datatype consists of the
following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| globalAggregate | number          | No              | Temperature     |
| TemperatureMarg |                 |                 | margin in       |
| in              |                 |                 | Celsius         |
|                 |                 |                 | relative to a   |
|                 |                 |                 | throttling      |
|                 |                 |                 | thermal trip    |
|                 |                 |                 | point           |
+-----------------+-----------------+-----------------+-----------------+
| globalAggregate | string          | Yes             | Identifier for  |
| TemperatureMarg |                 |                 | the ipmi global |
| inIdentifier    |                 |                 | aggregate       |
|                 |                 |                 | temperature     |
|                 |                 |                 | margin metrics  |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiHsbp
^^^^^^^^^^^^^^^^^^

The ipmiHsbp datatype provides ipmi hot swap backplane power metrics; it
consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| hsbpIdentifier  | string          | Yes             | Identifier for  |
|                 |                 |                 | the hot swap    |
|                 |                 |                 | backplane power |
|                 |                 |                 | unit            |
+-----------------+-----------------+-----------------+-----------------+
| hsbpTemperature | number          | No              | Hot swap        |
|                 |                 |                 | backplane power |
|                 |                 |                 | temperature in  |
|                 |                 |                 | celsius         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiNic
^^^^^^^^^^^^^^^^^

The ipmiNic datatype provides network interface control care metrics; it
consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| nicIdentifier   | string          | Yes             | Identifier for  |
|                 |                 |                 | the network     |
|                 |                 |                 | interface       |
|                 |                 |                 | control card    |
+-----------------+-----------------+-----------------+-----------------+
| nicTemperature  | number          | No              | nic temperature |
|                 |                 |                 | in Celsius      |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiPowerSupply
^^^^^^^^^^^^^^^^^^^^^^^^^

The ipmiPowerSupply datatype provides ipmi power supply metrics; it
consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| powerSupplyCurr | number          | No              | Current output  |
| entOutput       |                 |                 | voltage as a    |
| Percent         |                 |                 | percentage of   |
|                 |                 |                 | the design      |
|                 |                 |                 | specified level |
+-----------------+-----------------+-----------------+-----------------+
| powerSupplyIden | string          | Yes             | Identifier for  |
| tifier          |                 |                 | the power       |
|                 |                 |                 | supply          |
+-----------------+-----------------+-----------------+-----------------+
| powerSupplyInpu | number          | No              | Input power in  |
| tPower          |                 |                 | watts           |
+-----------------+-----------------+-----------------+-----------------+
| powerSupplyTemp | number          | No              | Power supply    |
| erature         |                 |                 | temperature in  |
|                 |                 |                 | Celsius         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: ipmiProcessor
^^^^^^^^^^^^^^^^^^^^^^^

The ipmiProcessor datatype provides ipmi processor metrics; it consists
of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| processorDimmAg | processorDimm   | No              | Array of        |
| gregate         | AggregateTherma |                 | processorDimmAg |
| ThermalMarginAr | l               |                 | gregate         |
| ray             | Margin [ ]      |                 | ThermalMargin   |
|                 |                 |                 | objects         |
+-----------------+-----------------+-----------------+-----------------+
| processorDtsThe | number          | No              | Front panel     |
| rmalMargin      |                 |                 | temperature in  |
|                 |                 |                 | celsius         |
+-----------------+-----------------+-----------------+-----------------+
| processorIdenti | string          | Yes             | Identifier for  |
| fier            |                 |                 | the power       |
|                 |                 |                 | supply          |
+-----------------+-----------------+-----------------+-----------------+
| pprocessorTherm | number          | No              | Io module       |
| alControl       |                 |                 | temperatue in   |
| Percent         |                 |                 | celsius         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: latencyBucketMeasure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The latencyBucketMeasure datatype consists of the following fields which
describe the number of counts falling within a defined latency bucket:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| countsInTheBuck | number          | Yes             | Number of       |
| et              |                 |                 | counts falling  |
|                 |                 |                 | within a        |
|                 |                 |                 | defined latency |
|                 |                 |                 | bucket          |
+-----------------+-----------------+-----------------+-----------------+
| highEndOfLatenc | number          | No              | High end of     |
| yBucket         |                 |                 | bucket range    |
|                 |                 |                 | (typically in   |
|                 |                 |                 | ms)             |
+-----------------+-----------------+-----------------+-----------------+
| lowEndOfLatency | number          | No              | Low end of      |
| Bucket          |                 |                 | bucket range    |
|                 |                 |                 | (typically in   |
|                 |                 |                 | ms)             |
+-----------------+-----------------+-----------------+-----------------+

Datatype: load
^^^^^^^^^^^^^^

The load datatype provides metrics on system cpu and io utilization
obtained using /proc/loadavg; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| longTerm        | number          | No              | number of jobs  |
|                 |                 |                 | in the run      |
|                 |                 |                 | queue (state R, |
|                 |                 |                 | cpu             |
|                 |                 |                 | utilization) or |
|                 |                 |                 | waiting for     |
|                 |                 |                 | disk I/O (state |
|                 |                 |                 | D, io           |
|                 |                 |                 | utilization)    |
|                 |                 |                 | averaged over   |
|                 |                 |                 | 15 minutes      |
|                 |                 |                 | using           |
|                 |                 |                 | /proc/loadavg   |
+-----------------+-----------------+-----------------+-----------------+
| midTerm         | number          | No              | number of jobs  |
|                 |                 |                 | in the run      |
|                 |                 |                 | queue (state R, |
|                 |                 |                 | cpu             |
|                 |                 |                 | utilization) or |
|                 |                 |                 | waiting for     |
|                 |                 |                 | disk I/O (state |
|                 |                 |                 | D, io           |
|                 |                 |                 | utilization)    |
|                 |                 |                 | averaged over 5 |
|                 |                 |                 | minutes using   |
|                 |                 |                 | /proc/loadavg   |
+-----------------+-----------------+-----------------+-----------------+
| shortTerm       | number          | No              | number of jobs  |
|                 |                 |                 | in the run      |
|                 |                 |                 | queue (state R, |
|                 |                 |                 | cpu             |
|                 |                 |                 | utilization) or |
|                 |                 |                 | waiting for     |
|                 |                 |                 | disk I/O (state |
|                 |                 |                 | D, io           |
|                 |                 |                 | utilization)    |
|                 |                 |                 | averaged over 1 |
|                 |                 |                 | minute using    |
|                 |                 |                 | /proc/loadavg   |
+-----------------+-----------------+-----------------+-----------------+

Datatype: machineCheckException
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The machineCheckException datatype describes machine check exceptions;
it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| correctedMemory | number          | No              | Total hardware  |
| Errors          |                 |                 | errors that     |
|                 |                 |                 | were corrected  |
|                 |                 |                 | by the hardware |
|                 |                 |                 | (e.g. data      |
|                 |                 |                 | corruption      |
|                 |                 |                 | corrected via   |
|                 |                 |                 |  ECC) over the  |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval.           |
|                 |                 |                 | These errors do |
|                 |                 |                 | not require     |
|                 |                 |                 | immediate       |
|                 |                 |                 | software        |
|                 |                 |                 | actions, but    |
|                 |                 |                 | are still       |
|                 |                 |                 | reported for    |
|                 |                 |                 | accounting      |
|                 |                 |                 | and predictive  |
|                 |                 |                 | failure         |
|                 |                 |                 | analysis        |
+-----------------+-----------------+-----------------+-----------------+
| correctedMemory | number          | No              | Total hardware  |
| Errors          |                 |                 | errors that     |
| In1Hr           |                 |                 | were corrected  |
|                 |                 |                 | by the hardware |
|                 |                 |                 | over the last   |
|                 |                 |                 | one hour        |
+-----------------+-----------------+-----------------+-----------------+
| processIdentifi | string          | Yes             | processIdentifi |
| er              |                 |                 | er              |
+-----------------+-----------------+-----------------+-----------------+
| uncorrectedMemo | number          | No              | Total           |
| ryErrors        |                 |                 | uncorrected     |
|                 |                 |                 | hardware errors |
|                 |                 |                 | that were       |
|                 |                 |                 | detected by the |
|                 |                 |                 | hardware (e.g., |
|                 |                 |                 | causing data    |
|                 |                 |                 | corruption)     |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval.           |
|                 |                 |                 | These errors    |
|                 |                 |                 | require a       |
|                 |                 |                 | software        |
|                 |                 |                 | response.       |
+-----------------+-----------------+-----------------+-----------------+
| uncorrectedMemo | number          | No              | Total           |
| ryErrors        |                 |                 | uncorrected     |
| In1Hr           |                 |                 | hardware errors |
|                 |                 |                 | that were       |
|                 |                 |                 | detected by the |
|                 |                 |                 | hardware over   |
|                 |                 |                 | the last one    |
|                 |                 |                 | hour            |
+-----------------+-----------------+-----------------+-----------------+

Datatype: measurementFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The measurementFields datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | measurement     |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| additionalMeasu | arrayOfNamedHas | No              | Array of named  |
| rements         | hMap            |                 | hashMap if      |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| additionalObjec | arrayOfJsonObje | No              | Array of JSON   |
| ts              | ct              |                 | objects         |
|                 |                 |                 | described by    |
|                 |                 |                 | name, schema    |
|                 |                 |                 | and other       |
|                 |                 |                 | meta-informatio |
|                 |                 |                 | n,              |
|                 |                 |                 | if needed       |
+-----------------+-----------------+-----------------+-----------------+
| codecUsageArray | codecsInUse []  | No              | Array of codecs |
|                 |                 |                 | in use          |
+-----------------+-----------------+-----------------+-----------------+
| concurrentSessi | integer         | No              | Peak concurrent |
| ons             |                 |                 | sessions for    |
|                 |                 |                 | the VM or xNF   |
|                 |                 |                 | (depending on   |
|                 |                 |                 | the context)    |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+
| configuredEntit | integer         | No              | Depending on    |
| ies             |                 |                 | the context     |
|                 |                 |                 | over the        |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval:           |
|                 |                 |                 | peak total      |
|                 |                 |                 | number of       |
|                 |                 |                 | users,          |
|                 |                 |                 | subscribers,    |
|                 |                 |                 | devices,        |
|                 |                 |                 | adjacencies,    |
|                 |                 |                 | etc., for the   |
|                 |                 |                 | VM, or peak     |
|                 |                 |                 | total number of |
|                 |                 |                 | subscribers,    |
|                 |                 |                 | devices, etc.,  |
|                 |                 |                 | for the xNF     |
+-----------------+-----------------+-----------------+-----------------+
| cpuUsageArray   | cpuUsage []     | No              | Usage of an     |
|                 |                 |                 | array of CPUs   |
+-----------------+-----------------+-----------------+-----------------+
| diskUsageArray  | diskUsage []    | No              | Usage of an     |
|                 |                 |                 | array of disks  |
+-----------------+-----------------+-----------------+-----------------+
| featureUsageArr | hashMap         | No              | The hashMap key |
| ay              |                 |                 | should identify |
|                 |                 |                 | the feature,    |
|                 |                 |                 | while the value |
|                 |                 |                 | defines the     |
|                 |                 |                 | number of times |
|                 |                 |                 | the identified  |
|                 |                 |                 | feature was     |
|                 |                 |                 | used            |
+-----------------+-----------------+-----------------+-----------------+
| filesystemUsage | filesystemUsage | No              | Filesystem      |
| Array           | [ ]             |                 | usage of the VM |
|                 |                 |                 | on which the    |
|                 |                 |                 | xNFC reporting  |
|                 |                 |                 | the event is    |
|                 |                 |                 | running         |
+-----------------+-----------------+-----------------+-----------------+
| hugePagesArray  | hugePages [ ]   | No              | Array of        |
|                 |                 |                 | metrics on      |
|                 |                 |                 | hugePages       |
+-----------------+-----------------+-----------------+-----------------+
| ipmiArray       | ipmi [ ]        | No              | Array of        |
|                 |                 |                 | intelligent     |
|                 |                 |                 | platform        |
|                 |                 |                 | management      |
|                 |                 |                 | interface       |
|                 |                 |                 | metrics         |
+-----------------+-----------------+-----------------+-----------------+
| latencyDistribu | latencyBucketMe | No              | Array of        |
| tion            | asure           |                 | integers        |
|                 | [ ]             |                 | representing    |
|                 |                 |                 | counts of       |
|                 |                 |                 | requests whose  |
|                 |                 |                 | latency in      |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | falls within    |
|                 |                 |                 | per-xNF         |
|                 |                 |                 | configured      |
|                 |                 |                 | ranges; where   |
|                 |                 |                 | latency is the  |
|                 |                 |                 | duration        |
|                 |                 |                 | between a       |
|                 |                 |                 | service request |
|                 |                 |                 | and its         |
|                 |                 |                 | fulfillment.    |
+-----------------+-----------------+-----------------+-----------------+
| loadArray       | load [ ]        | No              | Array of system |
|                 |                 |                 | load metrics    |
+-----------------+-----------------+-----------------+-----------------+
| machineCheckExc | machineCheckExc | No              | Array of        |
| eptionArray     | eption          |                 | machine check   |
|                 | [ ]             |                 | exceptions      |
+-----------------+-----------------+-----------------+-----------------+
| meanRequestLate | number          | No              | Mean seconds    |
| ncy             |                 |                 | required to     |
|                 |                 |                 | respond to each |
|                 |                 |                 | request for the |
|                 |                 |                 | VM on which the |
|                 |                 |                 | xNFC reporting  |
|                 |                 |                 | the event is    |
|                 |                 |                 | running         |
+-----------------+-----------------+-----------------+-----------------+
| measurementFiel | string          | Yes             | Version of the  |
| dsVersion       |                 |                 | measurementFiel |
|                 |                 |                 | ds              |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| measurementInte | number          | Yes             | Interval over   |
| rval            |                 |                 | which           |
|                 |                 |                 | measurements    |
|                 |                 |                 | are being       |
|                 |                 |                 | reported in     |
|                 |                 |                 | seconds         |
+-----------------+-----------------+-----------------+-----------------+
| memoryUsageArra | memoryUsage []  | No              | Memory usage of |
| y               |                 |                 | an array of VMs |
+-----------------+-----------------+-----------------+-----------------+
| nfcScalingMetri | integer         | No              | Represents      |
| c               |                 |                 | busy-ness of    |
|                 |                 |                 | the network     |
|                 |                 |                 | function from 0 |
|                 |                 |                 | to 100 as       |
|                 |                 |                 | reported by the |
|                 |                 |                 | nfc             |
+-----------------+-----------------+-----------------+-----------------+
| nicPerformanceA | nicPerformance  | No              | Performance     |
| rray            | [ ]             |                 | metrics of an   |
|                 |                 |                 | array of        |
|                 |                 |                 | network         |
|                 |                 |                 | interface cards |
+-----------------+-----------------+-----------------+-----------------+
| numberOfMediaPo | integer         | No              | Number of media |
| rtsInUse        |                 |                 | ports in use    |
+-----------------+-----------------+-----------------+-----------------+
| processStatsArr | processStats [  | No              | Array of        |
| ay              | ]               |                 | metrics on      |
|                 |                 |                 | system          |
|                 |                 |                 | processes       |
+-----------------+-----------------+-----------------+-----------------+
| requestRate     | number          | No              | Peak rate of    |
|                 |                 |                 | service         |
|                 |                 |                 | requests per    |
|                 |                 |                 | second to the   |
|                 |                 |                 | xNF over the    |
|                 |                 |                 | measurementInte |
|                 |                 |                 | rval            |
+-----------------+-----------------+-----------------+-----------------+

Datatype: memoryUsage
^^^^^^^^^^^^^^^^^^^^^

The memoryUsage datatype defines the memory usage of a virtual machine
and consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| memoryBuffered  | number          | No              | Kibibytes of    |
|                 |                 |                 | temporary       |
|                 |                 |                 | storage for raw |
|                 |                 |                 | disk blocks     |
+-----------------+-----------------+-----------------+-----------------+
| memoryCached    | number          | No              | Kibibytes of    |
|                 |                 |                 | memory used for |
|                 |                 |                 | cache           |
+-----------------+-----------------+-----------------+-----------------+
| memoryConfigure | number          | No              | Kibibytes of    |
| d               |                 |                 | memory          |
|                 |                 |                 | configured in   |
|                 |                 |                 | the virtual     |
|                 |                 |                 | machine on      |
|                 |                 |                 | which the xNFC  |
|                 |                 |                 | reporting the   |
|                 |                 |                 | event is        |
|                 |                 |                 | running         |
+-----------------+-----------------+-----------------+-----------------+
| memoryDemand    | number          | No              | Host demand in  |
|                 |                 |                 | kibibytes       |
+-----------------+-----------------+-----------------+-----------------+
| memoryFree      | number          | Yes             | Kibibytes of    |
|                 |                 |                 | physical RAM    |
|                 |                 |                 | left unused by  |
|                 |                 |                 | the system      |
+-----------------+-----------------+-----------------+-----------------+
| memoryLatencyAv | number          | No              | Percentage of   |
| g               |                 |                 | time the VM is  |
|                 |                 |                 | waiting to      |
|                 |                 |                 | access swapped  |
|                 |                 |                 | or compressed   |
|                 |                 |                 | memory          |
+-----------------+-----------------+-----------------+-----------------+
| memorySharedAvg | number          | No              | Shared memory   |
|                 |                 |                 | in kilobytes    |
+-----------------+-----------------+-----------------+-----------------+
| memorySlabRecl  | number          | No              | The part of the |
|                 |                 |                 | slab that can   |
|                 |                 |                 | be reclaimed    |
|                 |                 |                 | such as caches  |
|                 |                 |                 | measured in     |
|                 |                 |                 | kibibytes       |
+-----------------+-----------------+-----------------+-----------------+
| memorySlabUnrec | number          | No              | The part of the |
| l               |                 |                 | slab that       |
|                 |                 |                 | cannot be       |
|                 |                 |                 | reclaimed even  |
|                 |                 |                 | when lacking    |
|                 |                 |                 | memory measure  |
|                 |                 |                 | in kibibytes    |
+-----------------+-----------------+-----------------+-----------------+
| memorySwapInAvg | number          | No              | Amount of       |
|                 |                 |                 | memory          |
|                 |                 |                 | swapped-in from |
|                 |                 |                 | host cache in   |
|                 |                 |                 | kibibytes       |
+-----------------+-----------------+-----------------+-----------------+
| memorySwapInRat | number          | No              | Rate at which   |
| eAvg            |                 |                 | memory is       |
|                 |                 |                 | swapped from    |
|                 |                 |                 | disk into       |
|                 |                 |                 | active memory   |
|                 |                 |                 | during the      |
|                 |                 |                 | interval in     |
|                 |                 |                 | kilobytes per   |
|                 |                 |                 | second          |
+-----------------+-----------------+-----------------+-----------------+
| memorySwapOutAv | number          | No              | Amount of       |
| g               |                 |                 | memory          |
|                 |                 |                 | swapped-out to  |
|                 |                 |                 | host cache in   |
|                 |                 |                 | kibibytes       |
+-----------------+-----------------+-----------------+-----------------+
| memorySwapOutRa | number          | No              | Rate at which   |
| teAvg           |                 |                 | memory is being |
|                 |                 |                 | swapped from    |
|                 |                 |                 | active memory   |
|                 |                 |                 | to disk during  |
|                 |                 |                 | the current     |
|                 |                 |                 | interval in     |
|                 |                 |                 | kilobytes per   |
|                 |                 |                 | second          |
+-----------------+-----------------+-----------------+-----------------+
| memorySwapUsedA | number          | No              | Space used for  |
| vg              |                 |                 | caching swapped |
|                 |                 |                 | pages in the    |
|                 |                 |                 | host cache in   |
|                 |                 |                 | kibibytes       |
+-----------------+-----------------+-----------------+-----------------+
| memoryUsed      | number          | Yes             | Total memory    |
|                 |                 |                 | minus the sum   |
|                 |                 |                 | of free,        |
|                 |                 |                 | buffered,       |
|                 |                 |                 | cached and slab |
|                 |                 |                 | memory measured |
|                 |                 |                 | in kibibytes    |
+-----------------+-----------------+-----------------+-----------------+
| percentMemoryUs | number          | No              | Percentage of   |
| age             |                 |                 | memory usage;   |
|                 |                 |                 | value =         |
|                 |                 |                 | (memoryUsed /   |
|                 |                 |                 | (memoryUsed +   |
|                 |                 |                 | memoryFree) x   |
|                 |                 |                 | 100 if          |
|                 |                 |                 | denomintor is   |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| vmIdentifier    | string          | Yes             | Virtual Machine |
|                 |                 |                 | identifier      |
|                 |                 |                 | associated with |
|                 |                 |                 | the memory      |
|                 |                 |                 | metrics         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: nicPerformance
^^^^^^^^^^^^^^^^^^^^^^^^

The nicPerformance datatype consists of the following fields which
describe the performance and errors of an of an identified virtual
network interface card:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| administrativeS | string          | No              | Administrative  |
| tate            |                 |                 | state: enum:    |
|                 |                 |                 | ‘inService’,    |
|                 |                 |                 | ‘outOfService’  |
+-----------------+-----------------+-----------------+-----------------+
| nicIdentifier   | string          | Yes             | Network         |
|                 |                 |                 | interface card  |
|                 |                 |                 | identifier      |
+-----------------+-----------------+-----------------+-----------------+
| operationalStat | string          | No              | Operational     |
| e               |                 |                 | state: enum:    |
|                 |                 |                 | ‘inService’,    |
|                 |                 |                 | ‘outOfService’  |
+-----------------+-----------------+-----------------+-----------------+
| receivedBroadca | number          | No              | Cumulative      |
| stPacketsAccumu |                 |                 | count of        |
| lated           |                 |                 | broadcast       |
|                 |                 |                 | packets         |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedBroadca | number          | No              | Count of        |
| stPacketsDelta  |                 |                 | broadcast       |
|                 |                 |                 | packets         |
|                 |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedDiscard | number          | No              | Cumulative      |
| edPacketsAccumu |                 |                 | count of        |
| lated           |                 |                 | discarded       |
|                 |                 |                 | packets         |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedDiscard | number          | No              | Count of        |
| edPacketsDelta  |                 |                 | discarded       |
|                 |                 |                 | packets         |
|                 |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedErrorPa | number          | No              | Cumulative      |
| cketsAccumulate |                 |                 | count of error  |
| d               |                 |                 | packets         |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedErrorPa | number          | No              | Count of error  |
| cketsDelta      |                 |                 | packets         |
|                 |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedMultica | number          | No              | Cumulative      |
| stPacketsAccumu |                 |                 | count of        |
| lated           |                 |                 | multicast       |
|                 |                 |                 | packets         |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedMultica | number          | No              | Count of        |
| stPacketsDelta  |                 |                 | multicast       |
|                 |                 |                 | packets         |
|                 |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedOctetsA | number          | No              | Cumulative      |
| ccumulated      |                 |                 | count of octets |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedOctetsD | number          | No              | Count of octets |
| elta            |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedPercent | number          | No              | Percentage of   |
| Discard         |                 |                 | discarded       |
|                 |                 |                 | packets         |
|                 |                 |                 | received; value |
|                 |                 |                 | =               |
|                 |                 |                 | (receivedDiscar |
|                 |                 |                 | dedPacketsDelta |
|                 |                 |                 | /               |
|                 |                 |                 | receivedTotalPa |
|                 |                 |                 | cketsDelta)     |
|                 |                 |                 | x 100, if       |
|                 |                 |                 | denominator is  |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| receivedPercent | number          | No              | Percentage of   |
| Error           |                 |                 | error packets   |
|                 |                 |                 | received; value |
|                 |                 |                 | =               |
|                 |                 |                 | (receivedErrorP |
|                 |                 |                 | acketsDelta     |
|                 |                 |                 | /               |
|                 |                 |                 | receivedTotalPa |
|                 |                 |                 | cketsDelta)     |
|                 |                 |                 | x 100, if       |
|                 |                 |                 | denominator is  |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| receivedTotalPa | number          | No              | Cumulative      |
| cketsAccumulate |                 |                 | count of all    |
| d               |                 |                 | packets         |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedTotalPa | number          | No              | Count of all    |
| cketsDelta      |                 |                 | packets         |
|                 |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedUnicast | number          | No              | Cumulative      |
| PacketsAccumula |                 |                 | count of        |
| ted             |                 |                 | unicast packets |
|                 |                 |                 | received as     |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedUnicast | number          | No              | Count of        |
| PacketsDelta    |                 |                 | unicast packets |
|                 |                 |                 | received within |
|                 |                 |                 | the measurement |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| receivedUtiliza | number          | No              | Percentage of   |
| tion            |                 |                 | utilization     |
|                 |                 |                 | received; value |
|                 |                 |                 | =               |
|                 |                 |                 | (receivedOctets |
|                 |                 |                 | Delta           |
|                 |                 |                 | / (speed x      |
|                 |                 |                 | (lastEpochMicro |
|                 |                 |                 | sec             |
|                 |                 |                 | -               |
|                 |                 |                 | startEpochMicro |
|                 |                 |                 | sec)))          |
|                 |                 |                 | x 100, if       |
|                 |                 |                 | denominator is  |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| speed           | number          | No              | Speed           |
|                 |                 |                 | configured in   |
|                 |                 |                 | mbps.           |
+-----------------+-----------------+-----------------+-----------------+
| transmittedBroa | number          | No              | Cumulative      |
| dcastPacketsAcc |                 |                 | count of        |
| umulated        |                 |                 | broadcast       |
|                 |                 |                 | packets         |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedBroa | number          | No              | Count of        |
| dcastPacketsDel |                 |                 | broadcast       |
| ta              |                 |                 | packets         |
|                 |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedDisc | number          | No              | Cumulative      |
| ardedPacketsAcc |                 |                 | count of        |
| umulated        |                 |                 | discarded       |
|                 |                 |                 | packets         |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedDisc | number          | No              | Count of        |
| ardedPacketsDel |                 |                 | discarded       |
| ta              |                 |                 | packets         |
|                 |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedErro | number          | No              | Cumulative      |
| rPacketsAccumul |                 |                 | count of error  |
| ated            |                 |                 | packets         |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedErro | number          | No              | Count of error  |
| rPacketsDelta   |                 |                 | packets         |
|                 |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedMult | number          | No              | Cumulative      |
| icastPacketsAcc |                 |                 | count of        |
| umulated        |                 |                 | multicast       |
|                 |                 |                 | packets         |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedMult | number          | No              | Count of        |
| icastPacketsDel |                 |                 | multicast       |
| ta              |                 |                 | packets         |
|                 |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedOcte | number          | No              | Cumulative      |
| tsAccumulated   |                 |                 | count of octets |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedOcte | number          | No              | Count of octets |
| tsDelta         |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedPerc | number          | No              | Percentage of   |
| entDiscard      |                 |                 | discarded       |
|                 |                 |                 | packets         |
|                 |                 |                 | transmitted;    |
|                 |                 |                 | value =         |
|                 |                 |                 | (transmittedDis |
|                 |                 |                 | cardedPacketsDe |
|                 |                 |                 | lta             |
|                 |                 |                 | /               |
|                 |                 |                 | transmittedTota |
|                 |                 |                 | lPacketsDelta)  |
|                 |                 |                 | x 100, if       |
|                 |                 |                 | denominator is  |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| transmittedPerc | number          | No              | Percentage of   |
| entError        |                 |                 | error packets   |
|                 |                 |                 | received; value |
|                 |                 |                 | =               |
|                 |                 |                 | (transmittedErr |
|                 |                 |                 | orPacketsDelta  |
|                 |                 |                 | /               |
|                 |                 |                 | transmittedTota |
|                 |                 |                 | lPacketsDelta)  |
|                 |                 |                 | x 100, if       |
|                 |                 |                 | denominator is  |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| transmittedTota | number          | No              | Cumulative      |
| lPacketsAccumul |                 |                 | count of all    |
| ated            |                 |                 | packets         |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedTota | number          | No              | Count of all    |
| lPacketsDelta   |                 |                 | packets         |
|                 |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedUnic | number          | No              | Cumulative      |
| astPacketsAccum |                 |                 | count of        |
| ulated          |                 |                 | unicast packets |
|                 |                 |                 | transmitted as  |
|                 |                 |                 | read at the end |
|                 |                 |                 | of the          |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedUnic | number          | No              | Count of        |
| astPacketsDelta |                 |                 | unicast packets |
|                 |                 |                 | transmitted     |
|                 |                 |                 | within the      |
|                 |                 |                 | measurement     |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| transmittedUtil | number          | No              | Percentage of   |
| ization         |                 |                 | utilization     |
|                 |                 |                 | transmitted;    |
|                 |                 |                 | value =         |
|                 |                 |                 | (transmittedOct |
|                 |                 |                 | etsDelta        |
|                 |                 |                 | / (speed x      |
|                 |                 |                 | (lastEpochMicro |
|                 |                 |                 | sec             |
|                 |                 |                 | -               |
|                 |                 |                 | startEpochMicro |
|                 |                 |                 | sec)))          |
|                 |                 |                 | x 100, if       |
|                 |                 |                 | denominator is  |
|                 |                 |                 | nonzero, or 0,  |
|                 |                 |                 | if otherwise.   |
+-----------------+-----------------+-----------------+-----------------+
| valuesAreSuspec | string          | Yes             | Enumeration:    |
| t               |                 |                 | ‘true’ or       |
|                 |                 |                 | ‘false’. If     |
|                 |                 |                 | ‘true’ then the |
|                 |                 |                 | vNicPerformance |
|                 |                 |                 | values are      |
|                 |                 |                 | likely          |
|                 |                 |                 | inaccurate due  |
|                 |                 |                 | to counter      |
|                 |                 |                 | overflow or     |
|                 |                 |                 | other           |
|                 |                 |                 | conditions.     |
+-----------------+-----------------+-----------------+-----------------+

Datatype: processorDimmAggregateThermalMargin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The processorDimmAggregateThermalMargin datatype provides intelligent
platform management interface (ipmi) processor dual inline memory module
aggregate thermal margin metrics; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| processorDimmAg | string          | Yes             | identifier for  |
| gregateThermal  |                 |                 | the aggregate   |
| MarginIdentifie |                 |                 | thermal margin  |
| r               |                 |                 | metrics from    |
|                 |                 |                 | the processor   |
|                 |                 |                 | dual inline     |
|                 |                 |                 | memory module   |
+-----------------+-----------------+-----------------+-----------------+
| thermalMargin   | number          | Yes             | the difference  |
|                 |                 |                 | between the     |
|                 |                 |                 | DIMM's current  |
|                 |                 |                 | temperature, in |
|                 |                 |                 | celsius, and    |
|                 |                 |                 | the DIMM's      |
|                 |                 |                 | throttling      |
|                 |                 |                 | thermal trip    |
|                 |                 |                 | point           |
+-----------------+-----------------+-----------------+-----------------+

Datatype: processStats
^^^^^^^^^^^^^^^^^^^^^^

The processStats datatype provides metrics on system processes; it
consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| forkRate        | number          | No              | The number of   |
|                 |                 |                 | threads created |
|                 |                 |                 | since the last  |
|                 |                 |                 | reboot          |
+-----------------+-----------------+-----------------+-----------------+
| processIdentifi | string          | Yes             | processIdentifi |
| er              |                 |                 | er              |
+-----------------+-----------------+-----------------+-----------------+
| psStateBlocked  | number          | No              | The number of   |
|                 |                 |                 | processes in a  |
|                 |                 |                 | blocked state   |
+-----------------+-----------------+-----------------+-----------------+
| psStatePaging   | number          | No              | The number of   |
|                 |                 |                 | processes in a  |
|                 |                 |                 | paging state    |
+-----------------+-----------------+-----------------+-----------------+
| psStateRunning  | number          | No              | The number of   |
|                 |                 |                 | processes in a  |
|                 |                 |                 | running state   |
+-----------------+-----------------+-----------------+-----------------+
| psStateSleeping | number          | No              | The number of   |
|                 |                 |                 | processes in a  |
|                 |                 |                 | sleeping state  |
+-----------------+-----------------+-----------------+-----------------+
| psStateStopped  | number          | No              | The number of   |
|                 |                 |                 | processes in a  |
|                 |                 |                 | stopped state   |
+-----------------+-----------------+-----------------+-----------------+
| psStateZombie   | number          | No              | The number of   |
|                 |                 |                 | processes in a  |
|                 |                 |                 | zombie state    |
+-----------------+-----------------+-----------------+-----------------+

‘Notification’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: notificationFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The notificationFields datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | notification    |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| arrayOfNamedHas | namedHashMap [  | No              | Array of named  |
| hMap            | ]               |                 | hashMaps        |
+-----------------+-----------------+-----------------+-----------------+
| changeContact   | string          | No              | Identifier for  |
|                 |                 |                 | a contact       |
|                 |                 |                 | related to the  |
|                 |                 |                 | change          |
+-----------------+-----------------+-----------------+-----------------+
| changeIdentifie | string          | Yes             | System or       |
| r               |                 |                 | session         |
|                 |                 |                 | identifier      |
|                 |                 |                 | associated with |
|                 |                 |                 | the change      |
+-----------------+-----------------+-----------------+-----------------+
| changeType      | string          | Yes             | Describes what  |
|                 |                 |                 | has changed for |
|                 |                 |                 | the entity, for |
|                 |                 |                 | example:        |
|                 |                 |                 | configuration   |
|                 |                 |                 | changed,        |
|                 |                 |                 | capability      |
|                 |                 |                 | added,          |
|                 |                 |                 | capability      |
|                 |                 |                 | removed…        |
+-----------------+-----------------+-----------------+-----------------+
| newState        | string          | No              | New state of    |
|                 |                 |                 | the entity, for |
|                 |                 |                 | example:        |
|                 |                 |                 | ‘inService’,    |
|                 |                 |                 | ‘maintenance’,  |
|                 |                 |                 | ‘outOfService’  |
+-----------------+-----------------+-----------------+-----------------+
| notificationFie | string          | Yes             | Version of the  |
| ldsVersion      |                 |                 | notificationFie |
|                 |                 |                 | lds             |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| oldState        | string          | No              | Previous state  |
|                 |                 |                 | of the entity,  |
|                 |                 |                 | for example:    |
|                 |                 |                 | ‘inService’,    |
|                 |                 |                 | ‘maintenance’,  |
|                 |                 |                 | ‘outOfService’  |
+-----------------+-----------------+-----------------+-----------------+
| stateInterface  | string          | No              | Card or port    |
|                 |                 |                 | name of the     |
|                 |                 |                 | entity that     |
|                 |                 |                 | changed state   |
+-----------------+-----------------+-----------------+-----------------+

‘Other’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: otherFields
^^^^^^^^^^^^^^^^^^^^^

The otherFields datatype defines fields for events belonging to the
'other' domain of the commonEventHeader domain enumeration; it consists
of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| arrayOfNamedHas | arrayOfNamedHas | No              | Array of named  |
| hMap            | hMap            |                 | hashMaps        |
+-----------------+-----------------+-----------------+-----------------+
| hashMap         | hashMap         | No              | Array of        |
|                 |                 |                 | name-value      |
|                 |                 |                 | pairs           |
+-----------------+-----------------+-----------------+-----------------+
| jsonObjects     | arrayOfJsonObje | No              | Array of JSON   |
|                 | ct              |                 | objects         |
|                 |                 |                 | described by    |
|                 |                 |                 | name, schema    |
|                 |                 |                 | and other       |
|                 |                 |                 | meta-informatio |
|                 |                 |                 | n               |
+-----------------+-----------------+-----------------+-----------------+
| otherFieldsVers | string          | Yes             | Version of the  |
| ion             |                 |                 | otherFields     |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+

‘pnfRegistration’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: pnfRegistrationFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The pnfRegistrationFields datatype defines fields for events belonging
to the 'pnfRegistration' domain of the commonEventHeader domain
enumeration; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | pnfRegistration |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| lastServiceDate | string          | No              | TS 32.692       |
|                 |                 |                 | dateOfLastServi |
|                 |                 |                 | ce              |
|                 |                 |                 | = date of last  |
|                 |                 |                 | service; e.g.   |
|                 |                 |                 | 15022017        |
+-----------------+-----------------+-----------------+-----------------+
| macAddress      | string          | No              | MAC address of  |
|                 |                 |                 | OAM interface   |
|                 |                 |                 | of the unit     |
+-----------------+-----------------+-----------------+-----------------+
| manufactureDate | string          | No              | TS 32.692       |
|                 |                 |                 | dateOfManufactu |
|                 |                 |                 | re              |
|                 |                 |                 | = manufacture   |
|                 |                 |                 | date of the     |
|                 |                 |                 | unit; 24032016  |
+-----------------+-----------------+-----------------+-----------------+
| modelNumber     | string          | No              | TS 32.692       |
|                 |                 |                 | versionNumber = |
|                 |                 |                 | version of the  |
|                 |                 |                 | unit from       |
|                 |                 |                 | vendor; e.g.    |
|                 |                 |                 | AJ02. Maps to   |
|                 |                 |                 | AAI equip-model |
+-----------------+-----------------+-----------------+-----------------+
| oamV4IpAddress  | string          | No              | IPv4 m-plane IP |
|                 |                 |                 | address to be   |
|                 |                 |                 | used by the     |
|                 |                 |                 | manager to      |
|                 |                 |                 | contact the PNF |
+-----------------+-----------------+-----------------+-----------------+
| oamV6IpAddress  | string          | No              | IPv6 m-plane IP |
|                 |                 |                 | address to be   |
|                 |                 |                 | used by the     |
|                 |                 |                 | manager to      |
|                 |                 |                 | contact the PNF |
+-----------------+-----------------+-----------------+-----------------+
| pnfRegistration | string          | Yes             | Version of the  |
| FieldsVersion   |                 |                 | registrationFie |
|                 |                 |                 | lds             |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| serialNumber    | string          | No              | TS 32.692       |
|                 |                 |                 | serialNumber =  |
|                 |                 |                 | serial number   |
|                 |                 |                 | of the unit;    |
|                 |                 |                 | e.g. 6061ZW3    |
+-----------------+-----------------+-----------------+-----------------+
| softwareVersion | string          | No              | TS 32.692       |
|                 |                 |                 | swName = active |
|                 |                 |                 | SW running on   |
|                 |                 |                 | the unit; e.g.  |
|                 |                 |                 | 5gDUv18.05.201  |
+-----------------+-----------------+-----------------+-----------------+
| unitFamily      | string          | No              | TS 32.692       |
|                 |                 |                 | vendorUnitFamil |
|                 |                 |                 | yType           |
|                 |                 |                 | = general type  |
|                 |                 |                 | of HW unit;     |
|                 |                 |                 | e.g. BBU        |
+-----------------+-----------------+-----------------+-----------------+
| unitType        | string          | No              | TS 32.692       |
|                 |                 |                 | vendorUnitTypeN |
|                 |                 |                 | umber           |
|                 |                 |                 | = vendor name   |
|                 |                 |                 | for the unit;   |
|                 |                 |                 | e.g. Airscale   |
+-----------------+-----------------+-----------------+-----------------+
| vendorName      | string          | No              | TS 32.692       |
|                 |                 |                 | vendorName =    |
|                 |                 |                 | name of         |
|                 |                 |                 | manufacturer;   |
|                 |                 |                 | e.g. Nokia.     |
|                 |                 |                 | Maps to AAI     |
|                 |                 |                 | equip-vendor    |
+-----------------+-----------------+-----------------+-----------------+

 ‘State Change’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: stateChangeFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The stateChangeFields datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | stateChange     |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| newState        | string          | Yes             | New state of    |
|                 |                 |                 | the entity:     |
|                 |                 |                 | ‘inService’,    |
|                 |                 |                 | ‘maintenance’,  |
|                 |                 |                 | ‘outOfService’  |
+-----------------+-----------------+-----------------+-----------------+
| oldState        | string          | Yes             | Previous state  |
|                 |                 |                 | of the entity:  |
|                 |                 |                 | ‘inService’,    |
|                 |                 |                 | ‘maintenance’,  |
|                 |                 |                 | ‘outOfService’  |
+-----------------+-----------------+-----------------+-----------------+
| stateChangeFiel | string          | Yes             | Version of the  |
| dsVersion       |                 |                 | stateChangeFiel |
|                 |                 |                 | ds              |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| stateInterface  | string          | Yes             | Card or port    |
|                 |                 |                 | name of the     |
|                 |                 |                 | entity that     |
|                 |                 |                 | changed state   |
+-----------------+-----------------+-----------------+-----------------+

‘Syslog’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: syslogFields
^^^^^^^^^^^^^^^^^^^^^^

The syslogFields datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | syslog fields   |
|                 |                 |                 | if needed Ex:   |
|                 |                 |                 | {“name1”:       |
|                 |                 |                 | ”value1”,       |
|                 |                 |                 | “name2:         |
|                 |                 |                 | “value2” … }    |
+-----------------+-----------------+-----------------+-----------------+
| eventSourceHost | string          | No              | Hostname of the |
|                 |                 |                 | device          |
+-----------------+-----------------+-----------------+-----------------+
| eventSourceType | string          | Yes             | Examples:       |
|                 |                 |                 | ‘other’,        |
|                 |                 |                 | ‘router’,       |
|                 |                 |                 | ‘switch’,       |
|                 |                 |                 | ‘host’, ‘card’, |
|                 |                 |                 | ‘port’,         |
|                 |                 |                 | ‘slotThreshold’ |
|                 |                 |                 | ,               |
|                 |                 |                 | ‘portThreshold’ |
|                 |                 |                 | ,               |
|                 |                 |                 | ‘virtualMachine |
|                 |                 |                 | ’,              |
|                 |                 |                 | ‘virtualNetwork |
|                 |                 |                 | Function’       |
+-----------------+-----------------+-----------------+-----------------+
| syslogFacility  | integer         | No              | Numeric code    |
|                 |                 |                 | from 0 to 23    |
|                 |                 |                 | for facility:   |
|                 |                 |                 |                 |
|                 |                 |                 | 0 kernel        |
|                 |                 |                 | messages        |
|                 |                 |                 |                 |
|                 |                 |                 | 1 user-level    |
|                 |                 |                 | messages        |
|                 |                 |                 |                 |
|                 |                 |                 | 2 mail system   |
|                 |                 |                 |                 |
|                 |                 |                 | 3 system        |
|                 |                 |                 | daemons         |
|                 |                 |                 |                 |
|                 |                 |                 | 4               |
|                 |                 |                 | security/author |
|                 |                 |                 | ization         |
|                 |                 |                 | messages        |
|                 |                 |                 |                 |
|                 |                 |                 | 5 messages      |
|                 |                 |                 | generated       |
|                 |                 |                 | internally by   |
|                 |                 |                 | syslogd         |
|                 |                 |                 |                 |
|                 |                 |                 | 6 line printer  |
|                 |                 |                 | subsystem       |
|                 |                 |                 |                 |
|                 |                 |                 | 7 network news  |
|                 |                 |                 | subsystem       |
|                 |                 |                 |                 |
|                 |                 |                 | 8 UUCP          |
|                 |                 |                 | subsystem       |
|                 |                 |                 |                 |
|                 |                 |                 | 9 clock daemon  |
|                 |                 |                 |                 |
|                 |                 |                 | 10              |
|                 |                 |                 | security/author |
|                 |                 |                 | ization         |
|                 |                 |                 | messages        |
|                 |                 |                 |                 |
|                 |                 |                 | 11 FTP daemon   |
|                 |                 |                 |                 |
|                 |                 |                 | 12 NTP          |
|                 |                 |                 | subsystem       |
|                 |                 |                 |                 |
|                 |                 |                 | 13 log audit    |
|                 |                 |                 |                 |
|                 |                 |                 | 14 log alert    |
|                 |                 |                 |                 |
|                 |                 |                 | 15 clock daemon |
|                 |                 |                 | (note 2)        |
|                 |                 |                 |                 |
|                 |                 |                 | 16 local use 0  |
|                 |                 |                 | (local0)        |
|                 |                 |                 |                 |
|                 |                 |                 | 17 local use 1  |
|                 |                 |                 | (local1)        |
|                 |                 |                 |                 |
|                 |                 |                 | 18 local use 2  |
|                 |                 |                 | (local2)        |
|                 |                 |                 |                 |
|                 |                 |                 | 19 local use 3  |
|                 |                 |                 | (local3)        |
|                 |                 |                 |                 |
|                 |                 |                 | 20 local use 4  |
|                 |                 |                 | (local4)        |
|                 |                 |                 |                 |
|                 |                 |                 | 21 local use 5  |
|                 |                 |                 | (local5)        |
|                 |                 |                 |                 |
|                 |                 |                 | 22 local use 6  |
|                 |                 |                 | (local6)        |
|                 |                 |                 |                 |
|                 |                 |                 | 23 local use 7  |
|                 |                 |                 | (local7 )       |
+-----------------+-----------------+-----------------+-----------------+
| syslogFieldsVer | string          | Yes             | Version of the  |
| sion            |                 |                 | syslogFields    |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| syslogMsg       | string          | Yes             | Syslog message  |
+-----------------+-----------------+-----------------+-----------------+
| syslogMsgHost   | string          | No              | Hostname parsed |
|                 |                 |                 | from non-VES    |
|                 |                 |                 | syslog message  |
+-----------------+-----------------+-----------------+-----------------+
| syslogPri       | integer         | No              | 0-192           |
|                 |                 |                 |                 |
|                 |                 |                 | Combined        |
|                 |                 |                 | Severity and    |
|                 |                 |                 | Facility (see   |
|                 |                 |                 | rfc5424)        |
+-----------------+-----------------+-----------------+-----------------+
| syslogProc      | string          | No              | Identifies the  |
|                 |                 |                 | application     |
|                 |                 |                 | that originated |
|                 |                 |                 | the message     |
+-----------------+-----------------+-----------------+-----------------+
| syslogProcId    | number          | No              | The process     |
|                 |                 |                 | number assigned |
|                 |                 |                 | by the OS when  |
|                 |                 |                 | the application |
|                 |                 |                 | was started     |
+-----------------+-----------------+-----------------+-----------------+
| syslogSData     | string          | No              | A <space>       |
|                 |                 |                 | separated list  |
|                 |                 |                 | of key=”value”  |
|                 |                 |                 | pairs following |
|                 |                 |                 | the rfc5424     |
|                 |                 |                 | standard for    |
|                 |                 |                 | SD-ELEMENT.     |
|                 |                 |                 |                 |
|                 |                 |                 | **Deprecated**  |
|                 |                 |                 |                 |
|                 |                 |                 | The entire      |
|                 |                 |                 | rfc5424         |
|                 |                 |                 | syslogSData     |
|                 |                 |                 | object,         |
|                 |                 |                 | including       |
|                 |                 |                 | square brackets |
|                 |                 |                 | [ ], SD-ID and  |
|                 |                 |                 | list of         |
|                 |                 |                 | SD-PARAMs       |
+-----------------+-----------------+-----------------+-----------------+
| syslogSdId      | string          | No              | 0-32 char in    |
|                 |                 |                 | format          |
|                 |                 |                 | name@number,    |
|                 |                 |                 |                 |
|                 |                 |                 | i.e.,           |
|                 |                 |                 | ourSDID@32473   |
+-----------------+-----------------+-----------------+-----------------+
| syslogSev       | string          | No              | Level-of-severi |
|                 |                 |                 | ty              |
|                 |                 |                 | text            |
|                 |                 |                 | enumeration     |
|                 |                 |                 | defined below:  |
|                 |                 |                 |                 |
|                 |                 |                 | Text Sev        |
|                 |                 |                 | Description     |
|                 |                 |                 |                 |
|                 |                 |                 | Emergency 0     |
|                 |                 |                 | system is       |
|                 |                 |                 | unusable        |
|                 |                 |                 |                 |
|                 |                 |                 | Alert 1 action  |
|                 |                 |                 | must be taken   |
|                 |                 |                 | immediately     |
|                 |                 |                 |                 |
|                 |                 |                 | Critical 2      |
|                 |                 |                 | critical        |
|                 |                 |                 | conditions      |
|                 |                 |                 |                 |
|                 |                 |                 | Error 3 error   |
|                 |                 |                 | conditions      |
|                 |                 |                 |                 |
|                 |                 |                 | Warning 4       |
|                 |                 |                 | warning         |
|                 |                 |                 | conditions      |
|                 |                 |                 |                 |
|                 |                 |                 | Notice 5 normal |
|                 |                 |                 | but significant |
|                 |                 |                 | condition       |
|                 |                 |                 |                 |
|                 |                 |                 | Info 6          |
|                 |                 |                 | Informational   |
|                 |                 |                 | messages        |
|                 |                 |                 |                 |
|                 |                 |                 | Debug 7         |
|                 |                 |                 | debug-level     |
|                 |                 |                 | messages        |
+-----------------+-----------------+-----------------+-----------------+
| syslogTag       | string          | Yes             | Also known as   |
|                 |                 |                 | MsgId. Brief    |
|                 |                 |                 | non-spaced text |
|                 |                 |                 | indicating the  |
|                 |                 |                 | type of message |
|                 |                 |                 | such as         |
|                 |                 |                 | ‘TCPOUT’ or     |
|                 |                 |                 | ‘BGP_STATUS_CHA |
|                 |                 |                 | NGE’;           |
|                 |                 |                 | ‘NILVALUE’      |
|                 |                 |                 | should be used  |
|                 |                 |                 | when no other   |
|                 |                 |                 | value can be    |
|                 |                 |                 | provided        |
+-----------------+-----------------+-----------------+-----------------+
| syslogTs        | string          | No              | Timestamp       |
|                 |                 |                 | parsed from     |
|                 |                 |                 | non-VES syslog  |
|                 |                 |                 | message         |
+-----------------+-----------------+-----------------+-----------------+
| syslogVer       | number          | No              | IANA assigned   |
|                 |                 |                 | version of the  |
|                 |                 |                 | syslog protocol |
|                 |                 |                 | specification:  |
|                 |                 |                 |                 |
|                 |                 |                 | 0: VES          |
|                 |                 |                 |                 |
|                 |                 |                 | 1: IANA RFC5424 |
+-----------------+-----------------+-----------------+-----------------+

Examples of syslogSData :

Preferred

   ts=”1985-04-12T23:20:50.52Z” tag=”BGP_NEIGHBOR_DOWN” msg=”The BGP
   session to neighbor 10.10.10.10 is down”

Deprecated

   [attinc@1234 ts=”1985-04-12T23:20:50.52Z” tag=”BGP_NEIGHBOR_DOWN”
   msg=”The BGP session to neighbor 10.10.10.10 is down”]

Syslog references:

https://tools.ietf.org/html/rfc5424#section-6

   https://www.iana.org/assignments/syslog-parameters/syslog-parameters.xhtml

 ‘Threshold Crossing Alert’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: counter
^^^^^^^^^^^^^^^^^

The counter datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| criticality     | string          | Yes             | Enumeration:    |
|                 |                 |                 | ‘CRIT’, ‘MAJ’   |
+-----------------+-----------------+-----------------+-----------------+
| hashMap         | hashMap         | Yes             | Key is the name |
|                 |                 |                 | of the counter  |
|                 |                 |                 | and value is    |
|                 |                 |                 | the current     |
|                 |                 |                 | value of the    |
|                 |                 |                 | counter         |
+-----------------+-----------------+-----------------+-----------------+
| threshholdCross | string          | Yes             | Last threshold  |
| ed              |                 |                 | that was        |
|                 |                 |                 | crossed         |
+-----------------+-----------------+-----------------+-----------------+

Datatype: thresholdCrossingAlertFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The thresholdCrossingAlertFields datatype consists of the following
fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | threshold       |
|                 |                 |                 | crossing alert  |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| additionalParam | counter [ ]     | Yes             | Array of        |
| eters           |                 |                 | performance     |
|                 |                 |                 | counters        |
+-----------------+-----------------+-----------------+-----------------+
| alertAction     | string          | Yes             | Enumeration:    |
|                 |                 |                 | ‘SET’, ‘CONT’,  |
|                 |                 |                 | ‘CLEAR’         |
+-----------------+-----------------+-----------------+-----------------+
| alertDescriptio | string          | Yes             | Unique short    |
| n               |                 |                 | alert           |
|                 |                 |                 | description     |
|                 |                 |                 | (e.g.,          |
|                 |                 |                 | NE-CPUMEM)      |
+-----------------+-----------------+-----------------+-----------------+
| alertType       | string          | Yes             | Enumeration:    |
|                 |                 |                 | ‘CARD-ANOMALY’, |
|                 |                 |                 | ‘INTERFACE-ANOM |
|                 |                 |                 | ALY’,           |
|                 |                 |                 | ELEMENT-ANOMALY |
|                 |                 |                 | ’,              |
|                 |                 |                 | ‘SERVICE-ANOMAL |
|                 |                 |                 | Y’              |
+-----------------+-----------------+-----------------+-----------------+
| alertValue      | string          | No              | Calculated API  |
|                 |                 |                 | value (if       |
|                 |                 |                 | applicable)     |
+-----------------+-----------------+-----------------+-----------------+
| associatedAlert | string [ ]      | No              | List of         |
| IdList          |                 |                 | eventIds        |
|                 |                 |                 | associated with |
|                 |                 |                 | the event being |
|                 |                 |                 | reported        |
+-----------------+-----------------+-----------------+-----------------+
| collectionTimes | string          | Yes             | Time when the   |
| tamp            |                 |                 | performance     |
|                 |                 |                 | collector       |
|                 |                 |                 | picked up the   |
|                 |                 |                 | data; with RFC  |
|                 |                 |                 | 2822 compliant  |
|                 |                 |                 | format: ‘Sat,   |
|                 |                 |                 | 13 Mar 2010     |
|                 |                 |                 | 11:29:05 -0800’ |
+-----------------+-----------------+-----------------+-----------------+
| dataCollector   | string          | No              | Specific        |
|                 |                 |                 | performance     |
|                 |                 |                 | collector       |
|                 |                 |                 | instance used   |
+-----------------+-----------------+-----------------+-----------------+
| elementType     | string          | No              | Type of network |
|                 |                 |                 | element         |
|                 |                 |                 | (internal AT&T  |
|                 |                 |                 | field)          |
+-----------------+-----------------+-----------------+-----------------+
| eventSeverity   | string          | Yes             | Event severity  |
|                 |                 |                 | or priority     |
|                 |                 |                 | enumeration:    |
|                 |                 |                 | ‘CRITICAL’,     |
|                 |                 |                 | ‘MAJOR’,        |
|                 |                 |                 | ‘MINOR’,        |
|                 |                 |                 | ‘WARNING’,      |
|                 |                 |                 | ‘NORMAL’        |
+-----------------+-----------------+-----------------+-----------------+
| eventStartTimes | string          | Yes             | Time closest to |
| tamp            |                 |                 | when the        |
|                 |                 |                 | measurement was |
|                 |                 |                 | made; with RFC  |
|                 |                 |                 | 2822 compliant  |
|                 |                 |                 | format: ‘Sat,   |
|                 |                 |                 | 13 Mar 2010     |
|                 |                 |                 | 11:29:05 -0800’ |
+-----------------+-----------------+-----------------+-----------------+
| interfaceName   | string          | No              | Physical or     |
|                 |                 |                 | logical port or |
|                 |                 |                 | card (if        |
|                 |                 |                 | applicable)     |
+-----------------+-----------------+-----------------+-----------------+
| networkService  | string          | No              | Network name    |
|                 |                 |                 | (internal AT&T  |
|                 |                 |                 | field)          |
+-----------------+-----------------+-----------------+-----------------+
| possibleRootCau | string          | No              | Reserved for    |
| se              |                 |                 | future use      |
+-----------------+-----------------+-----------------+-----------------+
| thresholdCrossi | string          | Yes             | Version of the  |
| ng              |                 |                 | thresholdCrossi |
| FieldsVersion   |                 |                 | ngAlertFields   |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+

Technology Specific Datatypes
-----------------------------

 ‘Mobile Flow’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: gtpPerFlowMetrics
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The gtpPerFlowMetrics datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| avgBitErrorRate | number          | Yes             | Average bit     |
|                 |                 |                 | error rate      |
+-----------------+-----------------+-----------------+-----------------+
| avgPacketDelayV | number          | Yes             | Average packet  |
| ariation        |                 |                 | delay variation |
|                 |                 |                 | or jitter in    |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | for received    |
|                 |                 |                 | packets:        |
|                 |                 |                 | Average         |
|                 |                 |                 | difference      |
|                 |                 |                 | between the     |
|                 |                 |                 | packet          |
|                 |                 |                 | timestamp and   |
|                 |                 |                 | time received   |
|                 |                 |                 | for all pairs   |
|                 |                 |                 | of consecutive  |
|                 |                 |                 | packets         |
+-----------------+-----------------+-----------------+-----------------+
| avgPacketLatenc | number          | Yes             | Average         |
| y               |                 |                 | delivery        |
|                 |                 |                 | latency         |
+-----------------+-----------------+-----------------+-----------------+
| avgReceiveThrou | number          | Yes             | Average receive |
| ghput           |                 |                 | throughput      |
+-----------------+-----------------+-----------------+-----------------+
| avgTransmitThro | number          | Yes             | Average         |
| ughput          |                 |                 | transmit        |
|                 |                 |                 | throughput      |
+-----------------+-----------------+-----------------+-----------------+
| durConnectionFa | number          | No              | Duration of     |
| iledStatus      |                 |                 | failed state in |
|                 |                 |                 | milliseconds,   |
|                 |                 |                 | computed as the |
|                 |                 |                 | cumulative time |
|                 |                 |                 | between a       |
|                 |                 |                 | failed echo     |
|                 |                 |                 | request and the |
|                 |                 |                 | next following  |
|                 |                 |                 | successful      |
|                 |                 |                 | error request,  |
|                 |                 |                 | over this       |
|                 |                 |                 | reporting       |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| durTunnelFailed | number          | No              | Duration of     |
| Status          |                 |                 | errored state,  |
|                 |                 |                 | computed as the |
|                 |                 |                 | cumulative time |
|                 |                 |                 | between a       |
|                 |                 |                 | tunnel error    |
|                 |                 |                 | indicator and   |
|                 |                 |                 | the next        |
|                 |                 |                 | following       |
|                 |                 |                 | non-errored     |
|                 |                 |                 | indicator, over |
|                 |                 |                 | this reporting  |
|                 |                 |                 | interval        |
+-----------------+-----------------+-----------------+-----------------+
| flowActivatedBy | string          | No              | Endpoint        |
|                 |                 |                 | activating the  |
|                 |                 |                 | flow            |
+-----------------+-----------------+-----------------+-----------------+
| flowActivationE | number          | Yes             | Time the        |
| poch            |                 |                 | connection is   |
|                 |                 |                 | activated in    |
|                 |                 |                 | the flow        |
|                 |                 |                 | (connection)    |
|                 |                 |                 | being reported  |
|                 |                 |                 | on, or          |
|                 |                 |                 | transmission    |
|                 |                 |                 | time of the     |
|                 |                 |                 | first packet if |
|                 |                 |                 | activation time |
|                 |                 |                 | is not          |
|                 |                 |                 | available       |
+-----------------+-----------------+-----------------+-----------------+
| flowActivationM | number          | Yes             | Integer         |
| icrosec         |                 |                 | microseconds    |
|                 |                 |                 | for the start   |
|                 |                 |                 | of the flow     |
|                 |                 |                 | connection      |
+-----------------+-----------------+-----------------+-----------------+
| flowActivationT | string          | No              | Time the        |
| ime             |                 |                 | connection is   |
|                 |                 |                 | activated in    |
|                 |                 |                 | the flow being  |
|                 |                 |                 | reported on, or |
|                 |                 |                 | transmission    |
|                 |                 |                 | time of the     |
|                 |                 |                 | first packet if |
|                 |                 |                 | activation time |
|                 |                 |                 | is not          |
|                 |                 |                 | available; with |
|                 |                 |                 | RFC 2822        |
|                 |                 |                 | compliant       |
|                 |                 |                 | format: ‘Sat,   |
|                 |                 |                 | 13 Mar 2010     |
|                 |                 |                 | 11:29:05 -0800’ |
+-----------------+-----------------+-----------------+-----------------+
| flowDeactivated | string          | No              | Endpoint        |
| By              |                 |                 | deactivating    |
|                 |                 |                 | the flow        |
+-----------------+-----------------+-----------------+-----------------+
| flowDeactivatio | number          | Yes             | Time for the    |
| nEpoch          |                 |                 | start of the    |
|                 |                 |                 | flow            |
|                 |                 |                 | connection, in  |
|                 |                 |                 | integer UTC     |
|                 |                 |                 | epoch time aka  |
|                 |                 |                 | UNIX time       |
+-----------------+-----------------+-----------------+-----------------+
| flowDeactivatio | number          | Yes             | Integer         |
| nMicrosec       |                 |                 | microseconds    |
|                 |                 |                 | for the start   |
|                 |                 |                 | of the flow     |
|                 |                 |                 | connection      |
+-----------------+-----------------+-----------------+-----------------+
| flowDeactivatio | string          | Yes             | Transmission    |
| nTime           |                 |                 | time of the     |
|                 |                 |                 | first packet in |
|                 |                 |                 | the flow        |
|                 |                 |                 | connection      |
|                 |                 |                 | being reported  |
|                 |                 |                 | on; with RFC    |
|                 |                 |                 | 2822 compliant  |
|                 |                 |                 | format: ‘Sat,   |
|                 |                 |                 | 13 Mar 2010     |
|                 |                 |                 | 11:29:05 -0800’ |
+-----------------+-----------------+-----------------+-----------------+
| flowStatus      | string          | Yes             | Connection      |
|                 |                 |                 | status at       |
|                 |                 |                 | reporting time  |
|                 |                 |                 | as a working /  |
|                 |                 |                 | inactive /      |
|                 |                 |                 | failed          |
|                 |                 |                 | indicator value |
+-----------------+-----------------+-----------------+-----------------+
| gtpConnectionSt | string          | No              | Current         |
| atus            |                 |                 | connection      |
|                 |                 |                 | state at        |
|                 |                 |                 | reporting time  |
+-----------------+-----------------+-----------------+-----------------+
| gtpTunnelStatus | string          | No              | Current tunnel  |
|                 |                 |                 | state at        |
|                 |                 |                 | reporting time  |
+-----------------+-----------------+-----------------+-----------------+
| ipTosCountList  | hashMap         | No              | Array of key:   |
|                 |                 |                 | value pairs     |
|                 |                 |                 | where the keys  |
|                 |                 |                 | are drawn from  |
|                 |                 |                 | the IP          |
|                 |                 |                 | Type-of-Service |
|                 |                 |                 | identifiers     |
|                 |                 |                 | which range     |
|                 |                 |                 | from '0' to     |
|                 |                 |                 | '255', and the  |
|                 |                 |                 | values are the  |
|                 |                 |                 | count of        |
|                 |                 |                 | packets that    |
|                 |                 |                 | had those ToS   |
|                 |                 |                 | identifiers in  |
|                 |                 |                 | the flow        |
+-----------------+-----------------+-----------------+-----------------+
| ipTosList       | string          | No              | Array of unique |
|                 |                 |                 | IP              |
|                 |                 |                 | Type-of-Service |
|                 |                 |                 | values observed |
|                 |                 |                 | in the flow     |
|                 |                 |                 | where values    |
|                 |                 |                 | range from '0'  |
|                 |                 |                 | to '255'        |
+-----------------+-----------------+-----------------+-----------------+
| largePacketRtt  | number          | No              | large packet    |
|                 |                 |                 | round trip time |
+-----------------+-----------------+-----------------+-----------------+
| largePacketThre | number          | No              | large packet    |
| shold           |                 |                 | threshold being |
|                 |                 |                 | applied         |
+-----------------+-----------------+-----------------+-----------------+
| maxPacketDelayV | number          | Yes             | Maximum packet  |
| ariation        |                 |                 | delay variation |
|                 |                 |                 | or jitter in    |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | for received    |
|                 |                 |                 | packets:        |
|                 |                 |                 | Maximum of the  |
|                 |                 |                 | difference      |
|                 |                 |                 | between the     |
|                 |                 |                 | packet          |
|                 |                 |                 | timestamp and   |
|                 |                 |                 | time received   |
|                 |                 |                 | for all pairs   |
|                 |                 |                 | of consecutive  |
|                 |                 |                 | packets         |
+-----------------+-----------------+-----------------+-----------------+
| maxReceiveBitRa | number          | No              | maximum receive |
| te              |                 |                 | bit rate"       |
+-----------------+-----------------+-----------------+-----------------+
| maxTransmitBitR | number          | No              | maximum         |
| ate             |                 |                 | transmit bit    |
|                 |                 |                 | rate            |
+-----------------+-----------------+-----------------+-----------------+
| mobileQciCosCou | hashMap         | No              | array of key:   |
| ntList          |                 |                 | value pairs     |
|                 |                 |                 | where the keys  |
|                 |                 |                 | are drawn from  |
|                 |                 |                 | LTE QCI or UMTS |
|                 |                 |                 | class of        |
|                 |                 |                 | service         |
|                 |                 |                 | strings, and    |
|                 |                 |                 | the values are  |
|                 |                 |                 | the count of    |
|                 |                 |                 | packets that    |
|                 |                 |                 | had those       |
|                 |                 |                 | strings in the  |
|                 |                 |                 | flow            |
+-----------------+-----------------+-----------------+-----------------+
| mobileQciCosLis | string          | No              | Array of unique |
| t               |                 |                 | LTE QCI or UMTS |
|                 |                 |                 | class-of-servic |
|                 |                 |                 | e               |
|                 |                 |                 | values observed |
|                 |                 |                 | in the flow     |
+-----------------+-----------------+-----------------+-----------------+
| numActivationFa | number          | Yes             | Number of       |
| ilures          |                 |                 | failed          |
|                 |                 |                 | activation      |
|                 |                 |                 | requests, as    |
|                 |                 |                 | observed by the |
|                 |                 |                 | reporting node  |
+-----------------+-----------------+-----------------+-----------------+
| numBitErrors    | number          | Yes             | number of       |
|                 |                 |                 | errored bits    |
+-----------------+-----------------+-----------------+-----------------+
| numBytesReceive | number          | Yes             | number of bytes |
| d               |                 |                 | received,       |
|                 |                 |                 | including       |
|                 |                 |                 | retransmissions |
+-----------------+-----------------+-----------------+-----------------+
| numBytesTransmi | number          | Yes             | number of bytes |
| tted            |                 |                 | transmitted,    |
|                 |                 |                 | including       |
|                 |                 |                 | retransmissions |
+-----------------+-----------------+-----------------+-----------------+
| numDroppedPacke | number          | Yes             | number of       |
| ts              |                 |                 | received        |
|                 |                 |                 | packets dropped |
|                 |                 |                 | due to errors   |
|                 |                 |                 | per virtual     |
|                 |                 |                 | interface       |
+-----------------+-----------------+-----------------+-----------------+
| numGtpEchoFailu | number          | No              | Number of Echo  |
| res             |                 |                 | request path    |
|                 |                 |                 | failures where  |
|                 |                 |                 | failed paths    |
|                 |                 |                 | are defined in  |
|                 |                 |                 | 3GPP TS 29.281  |
|                 |                 |                 | sec 7.2.1 and   |
|                 |                 |                 | 3GPP TS 29.060  |
|                 |                 |                 | sec. 11.2       |
+-----------------+-----------------+-----------------+-----------------+
| numGtpTunnelErr | number          | No              | Number of       |
| ors             |                 |                 | tunnel error    |
|                 |                 |                 | indications     |
|                 |                 |                 | where errors    |
|                 |                 |                 | are defined in  |
|                 |                 |                 | 3GPP TS 29.281  |
|                 |                 |                 | sec 7.3.1 and   |
|                 |                 |                 | 3GPP TS 29.060  |
|                 |                 |                 | sec. 11.1       |
+-----------------+-----------------+-----------------+-----------------+
| numHttpErrors   | number          | No              | Http error      |
|                 |                 |                 | count           |
+-----------------+-----------------+-----------------+-----------------+
| numL7BytesRecei | number          | Yes             | number of       |
| ved             |                 |                 | tunneled layer  |
|                 |                 |                 | 7 bytes         |
|                 |                 |                 | received,       |
|                 |                 |                 | including       |
|                 |                 |                 | retransmissions |
+-----------------+-----------------+-----------------+-----------------+
| numL7BytesTrans | number          | Yes             | number of       |
| mitted          |                 |                 | tunneled layer  |
|                 |                 |                 | 7 bytes         |
|                 |                 |                 | transmitted,    |
|                 |                 |                 | excluding       |
|                 |                 |                 | retransmissions |
+-----------------+-----------------+-----------------+-----------------+
| numLostPackets  | number          | Yes             | number of lost  |
|                 |                 |                 | packets         |
+-----------------+-----------------+-----------------+-----------------+
| numOutOfOrderPa | number          | Yes             | number of       |
| ckets           |                 |                 | out-of-order    |
|                 |                 |                 | packets         |
+-----------------+-----------------+-----------------+-----------------+
| numPacketErrors | number          | Yes             | number of       |
|                 |                 |                 | errored packets |
+-----------------+-----------------+-----------------+-----------------+
| numPacketsRecei | number          | Yes             | number of       |
| vedExclRetrans  |                 |                 | packets         |
|                 |                 |                 | received,       |
|                 |                 |                 | excluding       |
|                 |                 |                 | retransmission  |
+-----------------+-----------------+-----------------+-----------------+
| numPacketsRecei | number          | Yes             | number of       |
| vedInclRetrans  |                 |                 | packets         |
|                 |                 |                 | received,       |
|                 |                 |                 | including       |
|                 |                 |                 | retransmission  |
+-----------------+-----------------+-----------------+-----------------+
| numPacketsTrans | number          | Yes             | number of       |
| mittedInclRetra |                 |                 | packets         |
| ns              |                 |                 | transmitted,    |
|                 |                 |                 | including       |
|                 |                 |                 | retransmissions |
+-----------------+-----------------+-----------------+-----------------+
| numRetries      | number          | Yes             | number of       |
|                 |                 |                 | packet retrie   |
+-----------------+-----------------+-----------------+-----------------+
| numTimeouts     | number          | Yes             | number of       |
|                 |                 |                 | packet timeouts |
+-----------------+-----------------+-----------------+-----------------+
| numTunneledL7By | number          | Yes             | number of       |
| tesReceived     |                 |                 | tunneled layer  |
|                 |                 |                 | 7 bytes         |
|                 |                 |                 | received,       |
|                 |                 |                 | excluding       |
|                 |                 |                 | retransmissions |
+-----------------+-----------------+-----------------+-----------------+
| roundTripTime   | number          | Yes             | Round Trip time |
+-----------------+-----------------+-----------------+-----------------+
| tcpFlagCountLis | hashMap         | No              | Array of key:   |
| t               |                 |                 | value pairs     |
|                 |                 |                 | where the keys  |
|                 |                 |                 | are drawn from  |
|                 |                 |                 | TCP Flags and   |
|                 |                 |                 | the values are  |
|                 |                 |                 | the count of    |
|                 |                 |                 | packets that    |
|                 |                 |                 | had that TCP    |
|                 |                 |                 | Flag in the     |
|                 |                 |                 | flow            |
+-----------------+-----------------+-----------------+-----------------+
| tcpFlagList     | string          | No              | Array of unique |
|                 |                 |                 | TCP Flags       |
|                 |                 |                 | observed in the |
|                 |                 |                 | flow            |
+-----------------+-----------------+-----------------+-----------------+
| timeToFirstByte | number          | Yes             | Time in         |
|                 |                 |                 | milliseconds    |
|                 |                 |                 | between the     |
|                 |                 |                 | connection      |
|                 |                 |                 | activation and  |
|                 |                 |                 | first byte      |
|                 |                 |                 | received        |
+-----------------+-----------------+-----------------+-----------------+

Datatype: mobileFlowFields
^^^^^^^^^^^^^^^^^^^^^^^^^^

The mobileFlowFields datatype consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalField | hashMap         | No              | Additional      |
| s               |                 |                 | mobileFlow      |
|                 |                 |                 | fields if       |
|                 |                 |                 | needed          |
+-----------------+-----------------+-----------------+-----------------+
| applicationType | string          | No              | Application     |
|                 |                 |                 | type inferred   |
+-----------------+-----------------+-----------------+-----------------+
| appProtocolType | string          | No              | Application     |
|                 |                 |                 | protocol        |
+-----------------+-----------------+-----------------+-----------------+
| appProtocolVers | string          | No              | Application     |
| ion             |                 |                 | version         |
+-----------------+-----------------+-----------------+-----------------+
| cid             | string          | No              | Cell Id         |
+-----------------+-----------------+-----------------+-----------------+
| connectionType  | string          | No              | Abbreviation    |
|                 |                 |                 | referencing a   |
|                 |                 |                 | 3GPP reference  |
|                 |                 |                 | point e.g.,     |
|                 |                 |                 | S1-U, S11, etc  |
+-----------------+-----------------+-----------------+-----------------+
| ecgi            | string          | No              | Evolved Cell    |
|                 |                 |                 | Global Id       |
+-----------------+-----------------+-----------------+-----------------+
| flowDirection   | string          | Yes             | Flow direction, |
|                 |                 |                 | indicating if   |
|                 |                 |                 | the reporting   |
|                 |                 |                 | node is the     |
|                 |                 |                 | source of the   |
|                 |                 |                 | flow or         |
|                 |                 |                 | destination for |
|                 |                 |                 | the flow        |
+-----------------+-----------------+-----------------+-----------------+
| gtpPerFlowMetri | gtpPer          | Yes             | Mobility GTP    |
| cs              | FlowMetrics     |                 | Protocol per    |
|                 |                 |                 | flow metrics    |
+-----------------+-----------------+-----------------+-----------------+
| gtpProtocolType | string          | No              | GTP protocol    |
+-----------------+-----------------+-----------------+-----------------+
| gtpVersion      | string          | No              | GTP protocol    |
|                 |                 |                 | version         |
+-----------------+-----------------+-----------------+-----------------+
| httpHeader      | string          | No              | HTTP request    |
|                 |                 |                 | header, if the  |
|                 |                 |                 | flow connects   |
|                 |                 |                 | to a node       |
|                 |                 |                 | referenced by   |
|                 |                 |                 | HTTP            |
+-----------------+-----------------+-----------------+-----------------+
| imei            | string          | No              | IMEI for the    |
|                 |                 |                 | subscriber UE   |
|                 |                 |                 | used in this    |
|                 |                 |                 | flow, if the    |
|                 |                 |                 | flow connects   |
|                 |                 |                 | to a mobile     |
|                 |                 |                 | device          |
+-----------------+-----------------+-----------------+-----------------+
| imsi            | string          | No              | IMSI for the    |
|                 |                 |                 | subscriber UE   |
|                 |                 |                 | used in this    |
|                 |                 |                 | flow, if the    |
|                 |                 |                 | flow connects   |
|                 |                 |                 | to a mobile     |
|                 |                 |                 | device          |
+-----------------+-----------------+-----------------+-----------------+
| ipProtocolType  | string          | Yes             | IP protocol     |
|                 |                 |                 | type e.g., TCP, |
|                 |                 |                 | UDP, RTP...     |
+-----------------+-----------------+-----------------+-----------------+
| ipVersion       | string          | Yes             | IP protocol     |
|                 |                 |                 | version e.g.,   |
|                 |                 |                 | IPv4, IPv6      |
+-----------------+-----------------+-----------------+-----------------+
| lac             | string          | No              | Location area   |
|                 |                 |                 | code            |
+-----------------+-----------------+-----------------+-----------------+
| mcc             | string          | No              | Mobile country  |
|                 |                 |                 | code            |
+-----------------+-----------------+-----------------+-----------------+
| mnc             | string          | No              | Mobile network  |
|                 |                 |                 | code            |
+-----------------+-----------------+-----------------+-----------------+
| mobileFlowField | string          | Yes             | Version of the  |
| sVersion        |                 |                 | mobileFlowField |
|                 |                 |                 | s               |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| msisdn          | string          | No              | MSISDN for the  |
|                 |                 |                 | subscriber UE   |
|                 |                 |                 | used in this    |
|                 |                 |                 | flow, as an     |
|                 |                 |                 | integer, if the |
|                 |                 |                 | flow connects   |
|                 |                 |                 | to a mobile     |
|                 |                 |                 | device          |
+-----------------+-----------------+-----------------+-----------------+
| otherEndpointIp | string          | Yes             | IP address for  |
| Address         |                 |                 | the other       |
|                 |                 |                 | endpoint, as    |
|                 |                 |                 | used for the    |
|                 |                 |                 | flow being      |
|                 |                 |                 | reported on     |
+-----------------+-----------------+-----------------+-----------------+
| otherEndpointPo | integer         | Yes             | IP Port for the |
| rt              |                 |                 | reporting       |
|                 |                 |                 | entity, as used |
|                 |                 |                 | for the flow    |
|                 |                 |                 | being reported  |
|                 |                 |                 | on              |
+-----------------+-----------------+-----------------+-----------------+
| otherFunctional | string          | No              | Functional role |
| Role            |                 |                 | of the other    |
|                 |                 |                 | endpoint for    |
|                 |                 |                 | the flow being  |
|                 |                 |                 | reported on     |
|                 |                 |                 | e.g., MME,      |
|                 |                 |                 | S-GW, P-GW,     |
|                 |                 |                 | PCRF...         |
+-----------------+-----------------+-----------------+-----------------+
| rac             | string          | No              | Routing area    |
|                 |                 |                 | code            |
+-----------------+-----------------+-----------------+-----------------+
| radioAccessTech | string          | No              | Radio Access    |
| nology          |                 |                 | Technology      |
|                 |                 |                 | e.g., 2G, 3G,   |
|                 |                 |                 | LTE             |
+-----------------+-----------------+-----------------+-----------------+
| reportingEndpoi | string          | Yes             | IP address for  |
| ntIpAddr        |                 |                 | the reporting   |
|                 |                 |                 | entity, as used |
|                 |                 |                 | for the flow    |
|                 |                 |                 | being reported  |
|                 |                 |                 | on              |
+-----------------+-----------------+-----------------+-----------------+
| reportingEndpoi | integer         | Yes             | IP port for the |
| ntPort          |                 |                 | reporting       |
|                 |                 |                 | entity, as used |
|                 |                 |                 | for the flow    |
|                 |                 |                 | being reported  |
|                 |                 |                 | on              |
+-----------------+-----------------+-----------------+-----------------+
| sac             | string          | No              | Service area    |
|                 |                 |                 | code            |
+-----------------+-----------------+-----------------+-----------------+
| samplingAlgorit | integer         | No              | Integer         |
| hm              |                 |                 | identifier for  |
|                 |                 |                 | the sampling    |
|                 |                 |                 | algorithm or    |
|                 |                 |                 | rule being      |
|                 |                 |                 | applied in      |
|                 |                 |                 | calculating the |
|                 |                 |                 | flow metrics if |
|                 |                 |                 | metrics are     |
|                 |                 |                 | calculated      |
|                 |                 |                 | based on a      |
|                 |                 |                 | sample of       |
|                 |                 |                 | packets, or 0   |
|                 |                 |                 | if no sampling  |
|                 |                 |                 | is applied      |
+-----------------+-----------------+-----------------+-----------------+
| tac             | string          | No              | Transport area  |
|                 |                 |                 | code            |
+-----------------+-----------------+-----------------+-----------------+
| tunnelId        | string          | No              | Tunnel          |
|                 |                 |                 | identifier      |
+-----------------+-----------------+-----------------+-----------------+
| vlanId          | string          | No              | VLAN identifier |
|                 |                 |                 | used by this    |
|                 |                 |                 | flow            |
+-----------------+-----------------+-----------------+-----------------+

 ‘SipSignaling’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: sipSignalingFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The sipSignalingFields datatype communicates information about sip
signaling messages, parameters and signaling state; it consists of the
following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalInfor | hashMap         | No              | Additional      |
| mation          |                 |                 | sipSignaling    |
|                 |                 |                 | fields          |
+-----------------+-----------------+-----------------+-----------------+
| compressedSip   | string          | No              | The full SIP    |
|                 |                 |                 | request/respons |
|                 |                 |                 | e               |
|                 |                 |                 | including       |
|                 |                 |                 | headers and     |
|                 |                 |                 | bodies          |
+-----------------+-----------------+-----------------+-----------------+
| correlator      | string          | Yes             | Constant across |
|                 |                 |                 | all events on   |
|                 |                 |                 | this call       |
+-----------------+-----------------+-----------------+-----------------+
| localIpAddress  | string          | Yes             | Ip address on   |
|                 |                 |                 | xNF             |
+-----------------+-----------------+-----------------+-----------------+
| localPort       | string          | Yes             | Port on xNF     |
+-----------------+-----------------+-----------------+-----------------+
| remoteIpAddress | string          | Yes             | IP address of   |
|                 |                 |                 | peer endpoint   |
+-----------------+-----------------+-----------------+-----------------+
| remotePort      | string          | Yes             | Port of peer    |
|                 |                 |                 | endpoint        |
+-----------------+-----------------+-----------------+-----------------+
| sipSignalingFie | string          | Yes             | Version of the  |
| ldsVersion      |                 |                 | sipSignalingFie |
|                 |                 |                 | lds             |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+
| summarySip      | string          | No              | The SIP Method  |
|                 |                 |                 | or Response     |
|                 |                 |                 | (‘INVITE’, ‘200 |
|                 |                 |                 | OK’, ‘BYE’,     |
|                 |                 |                 | etc)            |
+-----------------+-----------------+-----------------+-----------------+
| vendorNfNameFie | vendorNfNameFie | Yes             | Vendor, NF and  |
| lds             | lds             |                 | nfModule names  |
+-----------------+-----------------+-----------------+-----------------+

 ‘Voice Quality’ Domain Datatypes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Datatype: endOfCallVqmSummaries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The endOfCallVqmSummaries datatype provides end of call voice quality
metrics; it consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| adjacencyName   | string          | Yes             | Adjacency name  |
+-----------------+-----------------+-----------------+-----------------+
| endpointAverage | number          | No              | Endpoint        |
| Jitter          |                 |                 | average jitter  |
+-----------------+-----------------+-----------------+-----------------+
| endpointDescrip | string          | Yes             | Enumeration:    |
| tion            |                 |                 | ‘Caller’,       |
|                 |                 |                 | ‘Callee’        |
+-----------------+-----------------+-----------------+-----------------+
| endpointMaxJitt | number          | No              | Endpoint        |
| er              |                 |                 | maximum jitter  |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpOcte | number          | No              | Endpoint RTP    |
| tsDiscarded     |                 |                 | octets          |
|                 |                 |                 | discarded       |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpOcte | number          | No              | Endpoint RTP    |
| tsLost          |                 |                 | octets lost     |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpOcte | number          | No              | Endpoint RTP    |
| tsReceived      |                 |                 | octets received |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpOcte | number          | No              | Endpoint RTP    |
| tsSent          |                 |                 | octets sent     |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpPack | number          | No              | Endpoint RTP    |
| etsDiscarded    |                 |                 | packets         |
|                 |                 |                 | discarded       |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpPack | number          | No              | Endpoint RTP    |
| etsLost         |                 |                 | packets lost    |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpPack | number          | No              | Endpoint RTP    |
| etsReceived     |                 |                 | packets         |
|                 |                 |                 | received        |
+-----------------+-----------------+-----------------+-----------------+
| endpointRtpPack | number          | No              | Endpoint RTP    |
| etsSent         |                 |                 | packets sent    |
+-----------------+-----------------+-----------------+-----------------+
| localAverageJit | number          | No              | Local average   |
| ter             |                 |                 | jitter          |
+-----------------+-----------------+-----------------+-----------------+
| localAverageJit | number          | No              | Local average   |
| terBufferDelay  |                 |                 | jitter buffer   |
|                 |                 |                 | delay           |
+-----------------+-----------------+-----------------+-----------------+
| localMaxJitter  | number          | No              | Local maximum   |
|                 |                 |                 | jitter          |
+-----------------+-----------------+-----------------+-----------------+
| localMaxJitterB | number          | No              | Local max       |
| ufferDelay      |                 |                 | jitter buffer   |
|                 |                 |                 | delay           |
+-----------------+-----------------+-----------------+-----------------+
| localRtpOctetsD | number          | No              | Local RTP       |
| iscarded        |                 |                 | octets          |
|                 |                 |                 | discarded       |
+-----------------+-----------------+-----------------+-----------------+
| localRtpOctetsL | number          | No              | Local RTP       |
| ost             |                 |                 | octets lost     |
+-----------------+-----------------+-----------------+-----------------+
| localRtpOctetsR | number          | No              | Local RTP       |
| eceived         |                 |                 | octets received |
+-----------------+-----------------+-----------------+-----------------+
| localRtpOctetsS | number          | No              | Local RTP       |
| ent             |                 |                 | octets sent     |
+-----------------+-----------------+-----------------+-----------------+
| localRtpPackets | number          | No              | Local RTP       |
| Discarded       |                 |                 | packets         |
|                 |                 |                 | discarded       |
+-----------------+-----------------+-----------------+-----------------+
| localRtpPackets | number          | No              | Local RTP       |
| Lost            |                 |                 | packets lost    |
+-----------------+-----------------+-----------------+-----------------+
| localRtpPackets | number          | No              | Local RTP       |
| Received        |                 |                 | packets         |
|                 |                 |                 | received        |
+-----------------+-----------------+-----------------+-----------------+
| localRtpPackets | number          | No              | Local RTP       |
| Sent            |                 |                 | packets sent    |
+-----------------+-----------------+-----------------+-----------------+
| mosCqe          | number          | No              | Decimal range   |
|                 |                 |                 | from 1 to 5 (1  |
|                 |                 |                 | decimal place)  |
+-----------------+-----------------+-----------------+-----------------+
| oneWayDelay     | number          | No              | one-way path    |
|                 |                 |                 | delay in        |
|                 |                 |                 | milliseconds    |
+-----------------+-----------------+-----------------+-----------------+
| packetLossPerce | number          | No              | Calculated      |
| nt              |                 |                 | percentage      |
|                 |                 |                 | packet loss     |
|                 |                 |                 | based on        |
|                 |                 |                 | endpoint RTP    |
|                 |                 |                 | packets lost    |
|                 |                 |                 | (as reported in |
|                 |                 |                 | RTCP) and local |
|                 |                 |                 | RTP packets     |
|                 |                 |                 | sent. Direction |
|                 |                 |                 | is based on     |
|                 |                 |                 | endpoint        |
|                 |                 |                 | description     |
|                 |                 |                 | (Caller,        |
|                 |                 |                 | Callee).        |
|                 |                 |                 | Decimal (2      |
|                 |                 |                 | decimal places) |
+-----------------+-----------------+-----------------+-----------------+
| rFactor         | number          | No              | rFactor from 0  |
|                 |                 |                 | to 100          |
+-----------------+-----------------+-----------------+-----------------+
| roundTripDelay  | number          | No              | Round trip      |
|                 |                 |                 | delay in        |
|                 |                 |                 | milliseconds    |
+-----------------+-----------------+-----------------+-----------------+

Datatype: voiceQualityFields
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The voiceQualityFields datatype provides statistics related to customer
facing voice products; consists of the following fields:

+-----------------+-----------------+-----------------+-----------------+
| Field           | Type            | Required?       | Description     |
+=================+=================+=================+=================+
| additionalInfor | hashMap         | No              | Additional      |
| mation          |                 |                 | voice quality   |
|                 |                 |                 | fields          |
+-----------------+-----------------+-----------------+-----------------+
| calleeSideCodec | string          | Yes             | Callee codec    |
|                 |                 |                 | for the call    |
+-----------------+-----------------+-----------------+-----------------+
| callerSideCodec | string          | Yes             | Caller codec    |
|                 |                 |                 | for the call    |
+-----------------+-----------------+-----------------+-----------------+
| correlator      | string          | Yes             | Constant across |
|                 |                 |                 | all events on   |
|                 |                 |                 | this call       |
+-----------------+-----------------+-----------------+-----------------+
| endOfCallVqmSum | endOfCallVqm    | No              | End of call     |
| maries          | Summaries       |                 | voice quality   |
|                 |                 |                 | metric          |
|                 |                 |                 | summaries       |
+-----------------+-----------------+-----------------+-----------------+
| phoneNumber     | string          | No              | Phone number    |
|                 |                 |                 | associated with |
|                 |                 |                 | the correlator  |
+-----------------+-----------------+-----------------+-----------------+
| midCallRtcp     | string          | Yes             | Base64 encoding |
|                 |                 |                 | of the binary   |
|                 |                 |                 | RTCP data       |
|                 |                 |                 | (excluding      |
|                 |                 |                 | Eth/IP/UDP      |
|                 |                 |                 | headers)        |
+-----------------+-----------------+-----------------+-----------------+
| vendorNfNameFie | vendorNfNameFie | Yes             | Vendor, NF and  |
| lds             | lds             |                 | nfModule names  |
+-----------------+-----------------+-----------------+-----------------+
| voiceQualityFie | string          | Yes             | Version of the  |
| ldsVersion      |                 |                 | voiceQualityFie |
|                 |                 |                 | lds             |
|                 |                 |                 | block as “#.#”  |
|                 |                 |                 | where # is a    |
|                 |                 |                 | digit; see      |
|                 |                 |                 | section 1 for   |
|                 |                 |                 | the correct     |
|                 |                 |                 | digits to use.  |
+-----------------+-----------------+-----------------+-----------------+

Exceptions
==========

RESTful Web Services Exceptions
-------------------------------

RESTful services generate and send exceptions to clients in response to
invocation errors. Exceptions send HTTP status codes (specified later in
this document for each operation). HTTP status codes may be followed by
an optional JSON exception structure described below. Two types of
exceptions may be defined: service exceptions and policy exceptions.

+-----------------+-----------------+-----------------+-----------------+
| **Field Name**  | **Data Type**   | **Required?**   | **Description** |
+=================+=================+=================+=================+
| messageId       | xs:string       | Yes             | Unique message  |
|                 |                 |                 | identifier of   |
|                 |                 |                 | the format      |
|                 |                 |                 | ‘ABCnnnn’ where |
|                 |                 |                 | ‘ABC’ is either |
|                 |                 |                 | ‘SVC’ for       |
|                 |                 |                 | Service         |
|                 |                 |                 | Exceptions or   |
|                 |                 |                 | ‘POL’ for       |
|                 |                 |                 | Policy          |
|                 |                 |                 | Exception.      |
|                 |                 |                 |                 |
|                 |                 |                 | Exception       |
|                 |                 |                 | numbers may be  |
|                 |                 |                 | in the range of |
|                 |                 |                 | 0001 to 9999    |
|                 |                 |                 | where :         |
|                 |                 |                 |                 |
|                 |                 |                 | -  0001 to 2999 |
|                 |                 |                 |    are defined  |
|                 |                 |                 |    by OMA (see  |
|                 |                 |                 |    OMA’s        |
|                 |                 |                 |    `Common      |
|                 |                 |                 |    definitions  |
|                 |                 |                 |    for RESTful  |
|                 |                 |                 |    Network      |
|                 |                 |                 |    APIs <http:/ |
|                 |                 |                 | /technical.open |
|                 |                 |                 | mobilealliance. |
|                 |                 |                 | org/Technical/r |
|                 |                 |                 | elease_program/ |
|                 |                 |                 | docs/REST_NetAP |
|                 |                 |                 | I_Common/V1_0-2 |
|                 |                 |                 | 0120417-C/OMA-T |
|                 |                 |                 | S-REST_NetAPI_C |
|                 |                 |                 | ommon-V1_0-2012 |
|                 |                 |                 | 0417-C.pdf>`__  |
|                 |                 |                 |    for details) |
|                 |                 |                 |                 |
|                 |                 |                 | -  3000-9999    |
|                 |                 |                 |    are          |
|                 |                 |                 |    available    |
|                 |                 |                 |    and          |
|                 |                 |                 |    undefined    |
+-----------------+-----------------+-----------------+-----------------+
| text            | xs:string       | Yes             | Message text,   |
|                 |                 |                 | with            |
|                 |                 |                 | replacement     |
|                 |                 |                 | variables       |
|                 |                 |                 | marked with %n, |
|                 |                 |                 | where n is an   |
|                 |                 |                 | index into the  |
|                 |                 |                 | list of         |
|                 |                 |                 | <variables>     |
|                 |                 |                 | elements,       |
|                 |                 |                 | starting at 1   |
+-----------------+-----------------+-----------------+-----------------+
| variables       | xs:string       | No              | List of zero or |
|                 | [0..unbounded]  |                 | more strings    |
|                 |                 |                 | that represent  |
|                 |                 |                 | the contents of |
|                 |                 |                 | the variables   |
|                 |                 |                 | used by the     |
|                 |                 |                 | message text.   |
+-----------------+-----------------+-----------------+-----------------+
| url             | xs:anyUrl       | No              | Hyperlink to a  |
|                 |                 |                 | detailed error  |
|                 |                 |                 | resource (e.g., |
|                 |                 |                 | an HTML page    |
|                 |                 |                 | for browser     |
|                 |                 |                 | user agents).   |
+-----------------+-----------------+-----------------+-----------------+

Service Exceptions
------------------

When a service is not able to process a request, and retrying the
request with the same information will also result in a failure, and the
issue is not related to a service policy issue, then the service will
issue a fault using the service exception fault message. Examples of
service exceptions include invalid input, lack of availability of a
required resource or a processing error.

A service exception uses the letters 'SVC' at the beginning of the
message identifier. ‘SVC’ service exceptions used by the VES Event
Listener API are defined below.

+-------------+-------------+-------------+-------------+-------------+
| *MessageId* | *Descriptio | *Text*      | *Variables* | *Parent     |
|             | n           |             |             | HTTP Code*  |
|             | / Comment*  |             |             |             |
+=============+=============+=============+=============+=============+
| SVC0001     | General     | <custom     | None        | 400         |
|             | service     | error       |             |             |
|             | error (see  | message>    |             |             |
|             | SVC2000)    |             |             |             |
+-------------+-------------+-------------+-------------+-------------+
| SVC0002     | Bad         | Invalid     | %1: message | 400         |
|             | parameter   | input value | part        |             |
|             |             | for message |             |             |
|             |             | part %1     |             |             |
+-------------+-------------+-------------+-------------+-------------+
| SVC1000     | No server   | No server   | None        | 500         |
|             | resources   | resources   |             |             |
|             |             | available   |             |             |
|             |             | to process  |             |             |
|             |             | the request |             |             |
+-------------+-------------+-------------+-------------+-------------+
| SVC2000     | More        | The         | %1: human   | 400         |
|             | elaborate   | following   | readable    |             |
|             | version of  | service     | description |             |
|             | SVC0001     | error       | of the      |             |
|             |             | occurred:   | error       |             |
|             |             | %1. Error   |             |             |
|             |             | code is %2. | %2: error   |             |
|             |             |             | code        |             |
+-------------+-------------+-------------+-------------+-------------+

..

   Table 1 - Service Exceptions

Policy Exceptions
-----------------

When a service is not able to complete because the request fails to meet
a policy criteria, then the service will issue a fault using the policy
exception fault message. To clarify how a policy exception differs from
a service exception, consider that all the input to an operation may be
valid as meeting the required input for the operation (thus no service
exception), but using that input in the execution of the service may
result in conditions that require the service not to complete. Examples
of policy exceptions include privacy violations, requests not permitted
under a governing service agreement or input content not acceptable to
the service provider.

A Policy Exception uses the letters 'POL' at the beginning of the
message identifier. ‘POL’ policy exceptions used by the VES Event
Listener API are defined below.

+-------------+-------------+-------------+-------------+-------------+
| *MessageId* | *Descriptio | *Text*      | *Variables* | *Parent     |
|             | n           |             |             | HTTP Code*  |
|             | / Comment*  |             |             |             |
+=============+=============+=============+=============+=============+
| POL0001     | General     | A policy    | None        | 401         |
|             | policy      | error       |             |             |
|             | error (see  | occurred.   |             |             |
|             | POL2000)    |             |             |             |
+-------------+-------------+-------------+-------------+-------------+
| POL1009     | User not    | User has    | None        | 401         |
|             | provisioned | not been    |             |             |
|             | for service | provisioned |             |             |
|             |             | for service |             |             |
+-------------+-------------+-------------+-------------+-------------+
| POL1010     | User        | User has    | None        | 401         |
|             | suspended   | been        |             |             |
|             | from        | suspended   |             |             |
|             | service     | from        |             |             |
|             |             | service     |             |             |
+-------------+-------------+-------------+-------------+-------------+
| POL2000     | More        | The         | %1: human   | 401         |
|             | elaborate   | following   | readable    |             |
|             | version of  | policy      | description |             |
|             | POL0001     | error       | of the      |             |
|             |             | occurred:   | error       |             |
|             |             | %1. Error   |             |             |
|             |             | code is %2. | %2: error   |             |
|             |             |             | code        |             |
+-------------+-------------+-------------+-------------+-------------+
| POL9003     | Message     | Message     | None        | 400         |
|             | size        | content     |             |             |
|             | exceeds     | size        |             |             |
|             | limit       | exceeds the |             |             |
|             |             | allowable   |             |             |
|             |             | limit       |             |             |
+-------------+-------------+-------------+-------------+-------------+

..

   Table 2 - Policy Exceptions

RESTful Web Services Definition
===============================

REST Operation Overview
-----------------------

REST Operation Summary 
~~~~~~~~~~~~~~~~~~~~~~~

+-----------------------+-----------------------+-----------------------+
| **Operation Action**  | **HTTP**              | **Resource URL        |
|                       |                       | relative to           |
|                       | **Verb**              | {ServerRoot}, which   |
|                       |                       | is defined in         |
|                       |                       | section** **3**       |
+-----------------------+-----------------------+-----------------------+
| publishAnyEvent       | POST                  | /eventListener/v{apiV |
|                       |                       | ersion}               |
+-----------------------+-----------------------+-----------------------+
| publishEventBatch     | POST                  | /eventListener/v{apiV |
|                       |                       | ersion}/eventBatch    |
+-----------------------+-----------------------+-----------------------+

Table 3 - REST Operation Summary

Api Versioning
~~~~~~~~~~~~~~

apiVersion is used to describe the major version number of the event
listener API (which is the same as the major version number of this
specification). When this number changes, the implication is: the new
major version will break clients of older major versions in some way, if
they try to use the new API without modification (e.g., unmodified v1
clients would not be able to use v2 without error).

The Event Listener shall provide the following HTTP headers in response
to all requests. Additionally, clients may populate these headers on
requests to indicate the specific version they are interested in.

-  X-MinorVersion: 0

-  X-PatchVersion: 1

-  X-LatestVersion: 7.0.1

If a client requests major version 5 (per the REST resource URL) and
does not specify the above headers, then they will be provided with the
latest patch version of 5.0.x. If the client wants a particular minor
version of major version 5, then they need to supply the X-MinorVersion
header with their request. For example, if they request major version 5
with X-MinorVersion: 4, they will get the latest patch version of 5.4,
which is 5.4.1

Buffering of Events 
~~~~~~~~~~~~~~~~~~~~

{ServerRoot} is defined in section 3 of this document, which defines the
REST resource URL. One or more FQDNs may be provisioned in an event
source when it is instantiated or updated. If an event source is unable
to reach any of the provisioned FQDNs, it should buffer the event data
specified below, up to a maximum of 1 hour, until a connection can be
established and the events can be successfully delivered to the VES
Event Listener service.

xNFs acting as event sources should not send syslog events to the VES
Event Listener during debug mode (which is controlled via the Netconf
management interface), but should store syslog events locally for
access, and possible FTP transfer, via the xNF console (e.g., command
line interface).

If the internal event source event buffer or local storage should
overflow, then the event source should send a Fault event, and should
discard events in a first-in, first-out (FIFO) manner (i.e., discard
oldest events first).

Message Size
~~~~~~~~~~~~

Message size should be limited to 2 megabytes of uncompressed text sent
as application/json.

Operation: publishAnyEvent
--------------------------

Functional Behavior
~~~~~~~~~~~~~~~~~~~

Allows authorized clients to publish any single event to the VES event
listener.

-  Supports only secure HTTPS (one way SSL) access.

-  Uses the HTTP verb POST

-  Supports JSON content types

-  Provides HTTP response codes as well as Service and Policy error
   messages

Call Flow
~~~~~~~~~

|image3|

Figure 2 - publishAnyEvent Call Flow

Input Parameters
~~~~~~~~~~~~~~~~

Header Fields (note: all parameter names shall be treated as
case-insensitive):

+-----------------+-----------------+-----------------+-----------------+
| **Parameter**   | **Data Type**   | **Required?**   | **Brief         |
|                 |                 |                 | description**   |
+-----------------+-----------------+-----------------+-----------------+
| Accept          | string          | No              | Determines the  |
|                 |                 |                 | format of the   |
|                 |                 |                 | body of the     |
|                 |                 |                 | response. Valid |
|                 |                 |                 | values are:     |
|                 |                 |                 |                 |
|                 |                 |                 | -  application/ |
|                 |                 |                 | json            |
+-----------------+-----------------+-----------------+-----------------+
| Authorization   | string          | Yes             | The username    |
|                 |                 |                 | and password    |
|                 |                 |                 | are formed into |
|                 |                 |                 | one string as   |
|                 |                 |                 | “username:passw |
|                 |                 |                 | ord”.           |
|                 |                 |                 | This string is  |
|                 |                 |                 | then Base64     |
|                 |                 |                 | encoded to      |
|                 |                 |                 | produce the     |
|                 |                 |                 | encoded         |
|                 |                 |                 | credential      |
|                 |                 |                 | which is        |
|                 |                 |                 | communicated in |
|                 |                 |                 | the header      |
|                 |                 |                 | after the       |
|                 |                 |                 | string          |
|                 |                 |                 | “Authorization: |
|                 |                 |                 | Basic “. See    |
|                 |                 |                 | examples below. |
|                 |                 |                 | If the          |
|                 |                 |                 | Authorization   |
|                 |                 |                 | header is       |
|                 |                 |                 | missing, then   |
|                 |                 |                 | an HTTP 400     |
|                 |                 |                 | Invalid Request |
|                 |                 |                 | message shall   |
|                 |                 |                 | be returned. If |
|                 |                 |                 | the string      |
|                 |                 |                 | supplied is     |
|                 |                 |                 | invalid, then   |
|                 |                 |                 | an HTTP 401     |
|                 |                 |                 | Unauthorized    |
|                 |                 |                 | message shall   |
|                 |                 |                 | be returned.    |
+-----------------+-----------------+-----------------+-----------------+
| Content-length  | integer         | No              | Note that       |
|                 |                 |                 | content length  |
|                 |                 |                 | is limited to   |
|                 |                 |                 | 2Megabyte.      |
+-----------------+-----------------+-----------------+-----------------+
| Content-type    | string          | Yes             | Must be set to  |
|                 |                 |                 | one of the      |
|                 |                 |                 | following       |
|                 |                 |                 | values:         |
|                 |                 |                 |                 |
|                 |                 |                 | -  application/ |
|                 |                 |                 | json            |
+-----------------+-----------------+-----------------+-----------------+
| X-MinorVersion  | integer         | No              | The minor       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API requested   |
|                 |                 |                 | by the client   |
+-----------------+-----------------+-----------------+-----------------+
| X-PatchVersion  | integer         | No              | The patch       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API requested   |
|                 |                 |                 | by the client   |
+-----------------+-----------------+-----------------+-----------------+
| X-LatestVersion | string          | No              | The full        |
|                 |                 |                 | version of the  |
|                 |                 |                 | API requested   |
|                 |                 |                 | by the client   |
|                 |                 |                 | expressed as    |
|                 |                 |                 | {major}.{minor} |
|                 |                 |                 | .{patch}        |
+-----------------+-----------------+-----------------+-----------------+

Body Fields:

+-----------------+-----------------+-----------------+-----------------+
| **Parameter**   | **Data Type**   | **Required?**   | **Brief         |
|                 |                 |                 | description**   |
+-----------------+-----------------+-----------------+-----------------+
| Event           | event           | Yes             | Contains the    |
|                 |                 |                 | JSON structure  |
|                 |                 |                 | of the common   |
|                 |                 |                 | event format.   |
+-----------------+-----------------+-----------------+-----------------+

Output Parameters
~~~~~~~~~~~~~~~~~

Header fields:

+-----------------+-----------------+-----------------+-----------------+
| **Parameter**   | **Data Type**   | **Required?**   | **Brief         |
|                 |                 |                 | description**   |
+-----------------+-----------------+-----------------+-----------------+
| Content-length  | integer         | No              | Used only in    |
|                 |                 |                 | error           |
|                 |                 |                 | conditions.     |
+-----------------+-----------------+-----------------+-----------------+
| Content-type    | string          | No              | Used only in    |
|                 |                 |                 | error           |
|                 |                 |                 | conditions      |
+-----------------+-----------------+-----------------+-----------------+
| Date            | datetime        | No              | Date time of    |
|                 |                 |                 | the response in |
|                 |                 |                 | GMT             |
+-----------------+-----------------+-----------------+-----------------+
| X-MinorVersion  | integer         | Yes             | The minor       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API service     |
+-----------------+-----------------+-----------------+-----------------+
| X-PatchVersion  | integer         | Yes             | The patch       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API service     |
+-----------------+-----------------+-----------------+-----------------+
| X-LatestVersion | string          | Yes             | The full        |
|                 |                 |                 | version of the  |
|                 |                 |                 | API service     |
|                 |                 |                 | expressed as    |
|                 |                 |                 | {major}.{minor} |
|                 |                 |                 | .{patch}        |
+-----------------+-----------------+-----------------+-----------------+

Body Fields (for success responses): no content is provided.

Body Fields (for error responses):

+---------------+---------------+------------------+--------------------------------+
| **Parameter** | **Data Type** | **Required?**    | **Brief description**          |
+---------------+---------------+------------------+--------------------------------+
| requestError  | requestError  | Yes (for errors) | Used only in error conditions. |
+---------------+---------------+------------------+--------------------------------+

HTTP Status Codes
~~~~~~~~~~~~~~~~~

+-----------------------+-----------------------+-----------------------+
| *Code*                | *Reason Phrase*       | *Description*         |
+=======================+=======================+=======================+
| 202                   | Accepted              | The request has been  |
|                       |                       | accepted for          |
|                       |                       | processing            |
+-----------------------+-----------------------+-----------------------+
| 400                   | Bad Request           | Many possible reasons |
|                       |                       | not specified by the  |
|                       |                       | other codes (e.g.,    |
|                       |                       | missing required      |
|                       |                       | parameters or         |
|                       |                       | incorrect format).    |
|                       |                       | The response body may |
|                       |                       | include a further     |
|                       |                       | exception code and    |
|                       |                       | text. HTTP 400 errors |
|                       |                       | may be mapped to      |
|                       |                       | SVC0001 (general      |
|                       |                       | service error),       |
|                       |                       | SVC0002 (bad          |
|                       |                       | parameter), SVC2000   |
|                       |                       | (general service      |
|                       |                       | error with details)   |
|                       |                       | or PO9003 (message    |
|                       |                       | content size exceeds  |
|                       |                       | the allowable limit). |
+-----------------------+-----------------------+-----------------------+
| 401                   | Unauthorized          | Authentication failed |
|                       |                       | or was not provided.  |
|                       |                       | HTTP 401 errors may   |
|                       |                       | be mapped to POL0001  |
|                       |                       | (general policy       |
|                       |                       | error) or POL2000     |
|                       |                       | (general policy error |
|                       |                       | with details).        |
+-----------------------+-----------------------+-----------------------+
| 404                   | Not Found             | The server has not    |
|                       |                       | found anything        |
|                       |                       | matching the          |
|                       |                       | Request-URI. No       |
|                       |                       | indication is given   |
|                       |                       | of whether the        |
|                       |                       | condition is          |
|                       |                       | temporary or          |
|                       |                       | permanent.            |
+-----------------------+-----------------------+-----------------------+
| 405                   | Method Not Allowed    | A request was made of |
|                       |                       | a resource using a    |
|                       |                       | request method not    |
|                       |                       | supported by that     |
|                       |                       | resource (e.g., using |
|                       |                       | PUT on a REST         |
|                       |                       | resource that only    |
|                       |                       | supports POST).       |
+-----------------------+-----------------------+-----------------------+
| 500                   | Internal Server Error | The server            |
|                       |                       | encountered an        |
|                       |                       | internal error or     |
|                       |                       | timed out; please     |
|                       |                       | retry (general        |
|                       |                       | catch-all server-side |
|                       |                       | error).HTTP 500       |
|                       |                       | errors may be mapped  |
|                       |                       | to SVC1000 (no server |
|                       |                       | resources).           |
+-----------------------+-----------------------+-----------------------+

.. _sample-request-and-response-1:

Sample Request and Response
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _sample-request-1:

Sample Request
^^^^^^^^^^^^^^

+-----------------------------------------------------------------------+
| POST /eventListener/v7 HTTP/1.1                                       |
|                                                                       |
| Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==                     |
|                                                                       |
| content-type: application/json                                        |
|                                                                       |
| content-length: 12345                                                 |
|                                                                       |
| {                                                                     |
|                                                                       |
| "event": {                                                            |
|                                                                       |
|  "commonEventHeader": {                                               |
|                                                                       |
| "version": "4.0.1",                                                   |
|                                                                       |
| "vesEventListenerVersion": "7.0.1",                                   |
|                                                                       |
| "domain": "fault",                                                    |
|                                                                       |
| "eventName": "Fault_Vscf:Acs-Ericcson_PilotNumberPoolExhaustion",     |
|                                                                       |
| "eventId": "fault0000245",                                            |
|                                                                       |
| "sequence": 1,                                                        |
|                                                                       |
| "priority": "High",                                                   |
|                                                                       |
| "reportingEntityId": "cc305d54-75b4-431b-adb2-eb6b9e541234",          |
|                                                                       |
| "reportingEntityName": "ibcx0001vm002oam001",                         |
|                                                                       |
| "sourceId": "de305d54-75b4-431b-adb2-eb6b9e546014",                   |
|                                                                       |
| "sourceName": "scfx0001vm002cap001",                                  |
|                                                                       |
| "nfVendorName": "Ericsson",                                           |
|                                                                       |
| "nfNamingCode": "scfx",                                               |
|                                                                       |
| "nfcNamingCode": "ssc",                                               |
|                                                                       |
| "startEpochMicrosec": 1413378172000000,                               |
|                                                                       |
| "lastEpochMicrosec": 1413378172000000,                                |
|                                                                       |
| "timeZoneOffset": "UTC-05:30"                                         |
|                                                                       |
| },                                                                    |
|                                                                       |
| "faultFields": {                                                      |
|                                                                       |
| "faultFieldsVersion": 4.0,                                            |
|                                                                       |
| "alarmCondition": "PilotNumberPoolExhaustion",                        |
|                                                                       |
| "eventSourceType": "other",                                           |
|                                                                       |
| "specificProblem": "Calls cannot complete - pilot numbers are         |
| unavailable",                                                         |
|                                                                       |
| "eventSeverity": "CRITICAL",                                          |
|                                                                       |
| "vfStatus": "Active",                                                 |
|                                                                       |
| "alarmAdditionalInformation": {                                       |
|                                                                       |
| "PilotNumberPoolSize": "1000"                                         |
|                                                                       |
| }                                                                     |
|                                                                       |
| }                                                                     |
|                                                                       |
| }                                                                     |
|                                                                       |
| }                                                                     |
+-----------------------------------------------------------------------+

.. _sample-success-response-1:

Sample Success Response
^^^^^^^^^^^^^^^^^^^^^^^

+------------------------+
| HTTPS/1.1 202 Accepted |
|                        |
| X-MinorVersion: 0      |
|                        |
| X-PatchVersion: 1      |
|                        |
| X-LatestVersion: 7.0.1 |
+------------------------+

Sample Error Responses
^^^^^^^^^^^^^^^^^^^^^^

Sample Policy Exception
'''''''''''''''''''''''

+-------------------------------------------------------------+
| HTTPS/1.1 400 Bad Request                                   |
|                                                             |
| content-type: application/json                              |
|                                                             |
| content-length: 12345                                       |
|                                                             |
| Date: Thu, 04 Jun 2009 02:51:59 GMT                         |
|                                                             |
| X-MinorVersion: 0                                           |
|                                                             |
| X-PatchVersion: 1                                           |
|                                                             |
| X-LatestVersion: 7.0.1                                      |
|                                                             |
| {                                                           |
|                                                             |
| “requestError”: {                                           |
|                                                             |
| “policyException”: {                                        |
|                                                             |
| “messageId”: “POL9003”,                                     |
|                                                             |
| “text”: “Message content size exceeds the allowable limit”, |
|                                                             |
| }                                                           |
|                                                             |
| }                                                           |
|                                                             |
| }                                                           |
+-------------------------------------------------------------+

Sample Service Exception
''''''''''''''''''''''''

+---------------------------------------------------+
| HTTPS/1.1 400 Bad Request                         |
|                                                   |
| content-type: application/json                    |
|                                                   |
| content-length: 12345                             |
|                                                   |
| Date: Thu, 04 Jun 2009 02:51:59 GMT               |
|                                                   |
| X-MinorVersion: 0                                 |
|                                                   |
| X-PatchVersion: 1                                 |
|                                                   |
| X-LatestVersion: 7.0.1                            |
|                                                   |
| {                                                 |
|                                                   |
| “requestError”: {                                 |
|                                                   |
| “serviceException”: {                             |
|                                                   |
| “messageId”: “SVC2000”,                           |
|                                                   |
| “text”: “Missing Parameter: %1. Error code is %2” |
|                                                   |
| “variables”: [                                    |
|                                                   |
| “severity”,                                       |
|                                                   |
| “400”                                             |
|                                                   |
| ]                                                 |
|                                                   |
| }                                                 |
|                                                   |
| }                                                 |
|                                                   |
| }                                                 |
+---------------------------------------------------+

Operation: publishEventBatch
----------------------------

.. _functional-behavior-1:

Functional Behavior
~~~~~~~~~~~~~~~~~~~

Allows authorized clients to publish a batch of events to the VES event
listener.

-  Supports only secure HTTPS (one way SSL) access.

-  Uses the HTTP verb POST

-  Supports JSON content types

-  Provides HTTP response codes as well as Service and Policy error
   messages

.. _call-flow-1:

Call Flow
~~~~~~~~~

|image4|

Figure 3 – publishEventBatch Call Flow

.. _input-parameters-1:

Input Parameters
~~~~~~~~~~~~~~~~

Header Fields (note: all parameter names shall be treated as
case-insensitive):

+-----------------+-----------------+-----------------+-----------------+
| **Parameter**   | **Data Type**   | **Required?**   | **Brief         |
|                 |                 |                 | description**   |
+-----------------+-----------------+-----------------+-----------------+
| Accept          | string          | No              | Determines the  |
|                 |                 |                 | format of the   |
|                 |                 |                 | body of the     |
|                 |                 |                 | response. Valid |
|                 |                 |                 | values are:     |
|                 |                 |                 |                 |
|                 |                 |                 | -  application/ |
|                 |                 |                 | json            |
+-----------------+-----------------+-----------------+-----------------+
| Authorization   | string          | Yes             | The username    |
|                 |                 |                 | and password    |
|                 |                 |                 | are formed into |
|                 |                 |                 | one string as   |
|                 |                 |                 | “username:passw |
|                 |                 |                 | ord”.           |
|                 |                 |                 | This string is  |
|                 |                 |                 | then Base64     |
|                 |                 |                 | encoded to      |
|                 |                 |                 | produce the     |
|                 |                 |                 | encoded         |
|                 |                 |                 | credential      |
|                 |                 |                 | which is        |
|                 |                 |                 | communicated in |
|                 |                 |                 | the header      |
|                 |                 |                 | after the       |
|                 |                 |                 | string          |
|                 |                 |                 | “Authorization: |
|                 |                 |                 | Basic “. See    |
|                 |                 |                 | examples below. |
|                 |                 |                 | If the          |
|                 |                 |                 | Authorization   |
|                 |                 |                 | header is       |
|                 |                 |                 | missing, then   |
|                 |                 |                 | an HTTP 400     |
|                 |                 |                 | Invalid Request |
|                 |                 |                 | message shall   |
|                 |                 |                 | be returned. If |
|                 |                 |                 | the string      |
|                 |                 |                 | supplied is     |
|                 |                 |                 | invalid, then   |
|                 |                 |                 | an HTTP 401     |
|                 |                 |                 | Unauthorized    |
|                 |                 |                 | message shall   |
|                 |                 |                 | be returned.    |
+-----------------+-----------------+-----------------+-----------------+
| Content-length  | integer         | No              | Note that       |
|                 |                 |                 | content length  |
|                 |                 |                 | is limited to   |
|                 |                 |                 | 2Megabyte.      |
+-----------------+-----------------+-----------------+-----------------+
| Content-type    | string          | Yes             | Must be set to  |
|                 |                 |                 | one of the      |
|                 |                 |                 | following       |
|                 |                 |                 | values:         |
|                 |                 |                 |                 |
|                 |                 |                 | -  application/ |
|                 |                 |                 | json            |
+-----------------+-----------------+-----------------+-----------------+
| X-MinorVersion  | integer         | No              | The minor       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API requested   |
|                 |                 |                 | by the client   |
+-----------------+-----------------+-----------------+-----------------+
| X-PatchVersion  | integer         | No              | The patch       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API requested   |
|                 |                 |                 | by the client   |
+-----------------+-----------------+-----------------+-----------------+
| X-LatestVersion | string          | No              | The full        |
|                 |                 |                 | version of the  |
|                 |                 |                 | API requested   |
|                 |                 |                 | by the client   |
|                 |                 |                 | expressed as    |
|                 |                 |                 | {major}.{minor} |
|                 |                 |                 | .{patch}        |
+-----------------+-----------------+-----------------+-----------------+

Body Fields:

+-----------------+-----------------+-----------------+-----------------+
| **Parameter**   | **Data Type**   | **Required?**   | **Brief         |
|                 |                 |                 | description**   |
+-----------------+-----------------+-----------------+-----------------+
| eventList       | eventList       | Yes             | Array of events |
|                 |                 |                 | conforming to   |
|                 |                 |                 | the common      |
|                 |                 |                 | event format.   |
+-----------------+-----------------+-----------------+-----------------+

.. _output-parameters-1:

Output Parameters
~~~~~~~~~~~~~~~~~

Header fields:

+-----------------+-----------------+-----------------+-----------------+
| **Parameter**   | **Data Type**   | **Required?**   | **Brief         |
|                 |                 |                 | description**   |
+-----------------+-----------------+-----------------+-----------------+
| Content-length  | integer         | No              | Used only in    |
|                 |                 |                 | error           |
|                 |                 |                 | conditions.     |
+-----------------+-----------------+-----------------+-----------------+
| Content-type    | string          | No              | Used only in    |
|                 |                 |                 | error           |
|                 |                 |                 | conditions      |
+-----------------+-----------------+-----------------+-----------------+
| Date            | datetime        | No              | Date time of    |
|                 |                 |                 | the response in |
|                 |                 |                 | GMT             |
+-----------------+-----------------+-----------------+-----------------+
| X-MinorVersion  | integer         | Yes             | The minor       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API service     |
+-----------------+-----------------+-----------------+-----------------+
| X-PatchVersion  | integer         | Yes             | The patch       |
|                 |                 |                 | version of the  |
|                 |                 |                 | API service     |
+-----------------+-----------------+-----------------+-----------------+
| X-LatestVersion | string          | Yes             | The full        |
|                 |                 |                 | version of the  |
|                 |                 |                 | API service     |
|                 |                 |                 | expressed as    |
|                 |                 |                 | {major}.{minor} |
|                 |                 |                 | .{patch}        |
+-----------------+-----------------+-----------------+-----------------+

Body Fields (for success responses: no content is provided.

Body Fields (for error responses):

+---------------+---------------+------------------+--------------------------------+
| **Parameter** | **Data Type** | **Required?**    | **Brief description**          |
+---------------+---------------+------------------+--------------------------------+
| requestError  | requestError  | Yes (for errors) | Used only in error conditions. |
+---------------+---------------+------------------+--------------------------------+

.. _http-status-codes-1:

HTTP Status Codes
~~~~~~~~~~~~~~~~~

+-----------------------+-----------------------+-----------------------+
| *Code*                | *Reason Phrase*       | *Description*         |
+=======================+=======================+=======================+
| 202                   | Accepted              | The request has been  |
|                       |                       | accepted for          |
|                       |                       | processing            |
+-----------------------+-----------------------+-----------------------+
| 400                   | Bad Request           | Many possible reasons |
|                       |                       | not specified by the  |
|                       |                       | other codes (e.g.,    |
|                       |                       | missing required      |
|                       |                       | parameters or         |
|                       |                       | incorrect format).    |
|                       |                       | The response body may |
|                       |                       | include a further     |
|                       |                       | exception code and    |
|                       |                       | text. HTTP 400 errors |
|                       |                       | may be mapped to      |
|                       |                       | SVC0001 (general      |
|                       |                       | service error),       |
|                       |                       | SVC0002 (bad          |
|                       |                       | parameter), SVC2000   |
|                       |                       | (general service      |
|                       |                       | error with details)   |
|                       |                       | or PO9003 (message    |
|                       |                       | content size exceeds  |
|                       |                       | the allowable limit). |
+-----------------------+-----------------------+-----------------------+
| 401                   | Unauthorized          | Authentication failed |
|                       |                       | or was not provided.  |
|                       |                       | HTTP 401 errors may   |
|                       |                       | be mapped to POL0001  |
|                       |                       | (general policy       |
|                       |                       | error) or POL2000     |
|                       |                       | (general policy error |
|                       |                       | with details).        |
+-----------------------+-----------------------+-----------------------+
| 404                   | Not Found             | The server has not    |
|                       |                       | found anything        |
|                       |                       | matching the          |
|                       |                       | Request-URI. No       |
|                       |                       | indication is given   |
|                       |                       | of whether the        |
|                       |                       | condition is          |
|                       |                       | temporary or          |
|                       |                       | permanent.            |
+-----------------------+-----------------------+-----------------------+
| 405                   | Method Not Allowed    | A request was made of |
|                       |                       | a resource using a    |
|                       |                       | request method not    |
|                       |                       | supported by that     |
|                       |                       | resource (e.g., using |
|                       |                       | PUT on a REST         |
|                       |                       | resource that only    |
|                       |                       | supports POST).       |
+-----------------------+-----------------------+-----------------------+
| 500                   | Internal Server Error | The server            |
|                       |                       | encountered an        |
|                       |                       | internal error or     |
|                       |                       | timed out; please     |
|                       |                       | retry (general        |
|                       |                       | catch-all server-side |
|                       |                       | error).HTTP 500       |
|                       |                       | errors may be mapped  |
|                       |                       | to SVC1000 (no server |
|                       |                       | resources).           |
+-----------------------+-----------------------+-----------------------+

.. _sample-request-and-response-2:

Sample Request and Response
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _sample-request-2:

Sample Request
^^^^^^^^^^^^^^

+-----------------------------------------------------------------------+
| POST /eventListener/v7/eventBatch HTTP/1.1                            |
|                                                                       |
| Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==                     |
|                                                                       |
| content-type: application/json                                        |
|                                                                       |
| | content-length: 12345                                               |
| | {                                                                   |
|                                                                       |
| "eventList": [                                                        |
|                                                                       |
|  {                                                                    |
|                                                                       |
| "commonEventHeader": {                                                |
|                                                                       |
| "version": "4.0.1",                                                   |
|                                                                       |
| "vesEventListenerVersion": "7.0.1",                                   |
|                                                                       |
| "domain": "fault",                                                    |
|                                                                       |
| "eventName": "Fault_Vscf:Acs-Ericcson_PilotNumberPoolExhaustion",     |
|                                                                       |
| "eventId": "fault0000250",                                            |
|                                                                       |
| "sequence": 1,                                                        |
|                                                                       |
| "priority": "High",                                                   |
|                                                                       |
| "reportingEntityId": "cc305d54-75b4-431b-adb2-eb6b9e541234",          |
|                                                                       |
| "reportingEntityName": "ibcx0001vm002oam0011234",                     |
|                                                                       |
| "sourceId": "de305d54-75b4-431b-adb2-eb6b9e546014",                   |
|                                                                       |
| "sourceName": "scfx0001vm002cap001",                                  |
|                                                                       |
| "nfVendorName": "Ericsson",                                           |
|                                                                       |
| "nfNamingCode": "scfx",                                               |
|                                                                       |
| "nfcNamingCode": "ssc",                                               |
|                                                                       |
| "startEpochMicrosec": 1413378172000000,                               |
|                                                                       |
| "lastEpochMicrosec": 1413378172000000,                                |
|                                                                       |
| "timeZoneOffset": "UTC-05:30"                                         |
|                                                                       |
| },                                                                    |
|                                                                       |
| "faultFields": {                                                      |
|                                                                       |
| "faultFieldsVersion": 4.0,                                            |
|                                                                       |
| "alarmCondition": "PilotNumberPoolExhaustion",                        |
|                                                                       |
| "eventSourceType": "other",                                           |
|                                                                       |
| "specificProblem": "Calls cannot complete - pilot numbers are         |
| unavailable",                                                         |
|                                                                       |
| "eventSeverity": "CRITICAL",                                          |
|                                                                       |
| "vfStatus": "Active",                                                 |
|                                                                       |
| "alarmAdditionalInformation": {                                       |
|                                                                       |
| "PilotNumberPoolSize": "1000"                                         |
|                                                                       |
| }                                                                     |
|                                                                       |
| }                                                                     |
|                                                                       |
| },                                                                    |
|                                                                       |
| {                                                                     |
|                                                                       |
| "commonEventHeader": {                                                |
|                                                                       |
| "version": "4.0.1",                                                   |
|                                                                       |
| "vesEventListenerVersion": "7.0.1",                                   |
|                                                                       |
| "domain": "fault",                                                    |
|                                                                       |
| "eventName": " Fault_Vscf:Acs-Ericcson_RecordingServerUnreachable",   |
|                                                                       |
| "eventId": "fault0000251",                                            |
|                                                                       |
| "sequence": 0,                                                        |
|                                                                       |
| "priority": "High",                                                   |
|                                                                       |
| "reportingEntityId": "cc305d54-75b4-431b-adb2-eb6b9e541234",          |
|                                                                       |
| "reportingEntityName": "ibcx0001vm002oam0011234",                     |
|                                                                       |
| "sourceId": "de305d54-75b4-431b-adb2-eb6b9e546014",                   |
|                                                                       |
| "sourceName": "scfx0001vm002cap001",                                  |
|                                                                       |
| "nfVendorName": "Ericsson",                                           |
|                                                                       |
| "nfNamingCode": "scfx",                                               |
|                                                                       |
| "nfcNamingCode": "ssc",                                               |
|                                                                       |
| "startEpochMicrosec": 1413378172000010,                               |
|                                                                       |
| "lastEpochMicrosec": 1413378172000010,                                |
|                                                                       |
| "timeZoneOffset": "UTC-05:30"                                         |
|                                                                       |
| },                                                                    |
|                                                                       |
| "faultFields": {                                                      |
|                                                                       |
| "faultFieldsVersion": 4.0,                                            |
|                                                                       |
| "alarmCondition": "RecordingServerUnreachable",                       |
|                                                                       |
| "eventSourceType": "other",                                           |
|                                                                       |
| "specificProblem": "Recording server unreachable",                    |
|                                                                       |
| "eventSeverity": "CRITICAL",                                          |
|                                                                       |
| "vfStatus": "Active"                                                  |
|                                                                       |
| }                                                                     |
|                                                                       |
| }                                                                     |
|                                                                       |
| ]                                                                     |
|                                                                       |
| }                                                                     |
+-----------------------------------------------------------------------+

.. _sample-success-response-2:

Sample Success Response
^^^^^^^^^^^^^^^^^^^^^^^

+------------------------+
| HTTPS/1.1 202 Accepted |
|                        |
| X-MinorVersion: 0      |
|                        |
| X-PatchVersion: 1      |
|                        |
| X-LatestVersion: 7.0.1 |
+------------------------+

.. _sample-error-responses-1:

Sample Error Responses
^^^^^^^^^^^^^^^^^^^^^^

.. _sample-policy-exception-1:

Sample Policy Exception
'''''''''''''''''''''''

+-------------------------------------------------------------+
| HTTPS/1.1 400 Bad Request                                   |
|                                                             |
| content-type: application/json                              |
|                                                             |
| content-length: 12345                                       |
|                                                             |
| Date: Thu, 04 Jun 2009 02:51:59 GMT                         |
|                                                             |
| X-MinorVersion: 0                                           |
|                                                             |
| X-PatchVersion: 1                                           |
|                                                             |
| X-LatestVersion: 7.0.1                                      |
|                                                             |
| {                                                           |
|                                                             |
| “requestError”: {                                           |
|                                                             |
| “policyException”: {                                        |
|                                                             |
| “messageId”: “POL9003”,                                     |
|                                                             |
| “text”: “Message content size exceeds the allowable limit”, |
|                                                             |
| }                                                           |
|                                                             |
| }                                                           |
|                                                             |
| }                                                           |
+-------------------------------------------------------------+

.. _sample-service-exception-1:

Sample Service Exception
''''''''''''''''''''''''

+---------------------------------------------------+
| HTTPS/1.1 400 Bad Request                         |
|                                                   |
| content-type: application/json                    |
|                                                   |
| content-length: 12345                             |
|                                                   |
| Date: Thu, 04 Jun 2009 02:51:59 GMT               |
|                                                   |
| X-MinorVersion: 0                                 |
|                                                   |
| X-PatchVersion: 1                                 |
|                                                   |
| X-LatestVersion: 7.0.0                            |
|                                                   |
| {                                                 |
|                                                   |
| “requestError”: {                                 |
|                                                   |
| “serviceException”: {                             |
|                                                   |
| “messageId”: “SVC2000”,                           |
|                                                   |
| “text”: “Missing Parameter: %1. Error code is %2” |
|                                                   |
| “variables”: [                                    |
|                                                   |
| “severity”,                                       |
|                                                   |
| “400”                                             |
|                                                   |
| ]                                                 |
|                                                   |
| }                                                 |
|                                                   |
| }                                                 |
|                                                   |
| }                                                 |
+---------------------------------------------------+

Terminology
===========

Terminology used in this document is summarized below:

**A&AI**. Active & Available Inventory is the ONAP component that
provides data views of Customer Subscriptions, Products, Services,
Resources, and their relationships.

**Alarm Condition**. Short name of the alarm condition/problem, such as
a trap name.

**APPC (formerly APP-C)**. Application Controller. Handles the life
cycle management of Virtual Network Functions (VNFs).

**ASDC**. AT&T Service Design and Creation Platform: the original name
for the SDC. Replaced by SDC.

**Common Event Format**. A JSON schema describing events sent to the VES
Event Listener.

**Common Event Header**. A component of the Common Event Format JSON
structure. This datatype consists of fields common to all events.

**DCAE**. Data Collection Analysis and Events. DCAE is the ONAP
subsystem that supports closed loop control and higher-level correlation
for business and operations activities. DCAE collects performance,
usage, and configuration data, provides computation of analytics, aids
in trouble-shooting and management, and publishes event, data, and
analytics to the rest of the ONAP system for FCAPS functionality.

**DMaaP.** Data Movement as a Platform. A set of common services
provided by ONAP, including a Message Router, Data Router, and a Data
Bus Controller.

**Domain**. In VES, an event ‘domain’ identifies a broad category of
events (e.g., ‘fault’ or ‘measurement’), each of which is associated
with a VES domain field block, which is sent with the commonEventHeader
when events of that category are generated.

**ECOMP**. Enhanced Control, Orchestration, Management and Policy
preceded ONAP and is the name given to AT&T’s instance of the ONAP
platform.

**Epoch**. The number of seconds that have elapsed since
00:00:00 \ `Coordinated Universal
Time <https://en.wikipedia.org/wiki/Coordinated_Universal_Time>`__ (UTC),
Thursday, 1 January 1970. Every day is treated as if it contains exactly
86400 seconds, so \ `leap
seconds <https://en.wikipedia.org/wiki/Leap_second>`__ are not applied
to seconds since the Epoch. In VES Epoch times are measured in
microseconds.

**Event.** A well-structured packet of network management information
identified by an eventName which is asynchronously communicated to one
or more instances of an Event Listener service to subscribers interested
in that eventName. Events can convey measurements, faults, syslogs,
threshold crossing alerts, and others types of information.

**Event Id**. Event key that is unique to the event source. The key must
be unique within notification life cycle similar to EventID from 3GPP.
It could be a sequential number, or a composite key formed from the
event fields, such as sourceName_alarmCondition_startEpoch. The eventId
should not include whitespace. For fault events, eventId is the eventId
of the initial alarm; if the same alarm is raised again for changed,
acknowledged or cleared cases, eventId must be the same as the initial
alarm (along with the same startEpochMicrosec and an incremental
sequence number.

**Event Name**. Identifier for specific types of events. Specific
eventNames registered by the YAML may require that certain fields, which
are optional in the Common Event Format, be present when events with
that eventName are published.

**Event Streaming**. The delivery of network management event
information in real time.

**Extensible Data Structures**. Data structures (e.g., hashMap) that
allow event sources to send information not specifically identified in
the VES schema.

**Hash Map**. A hash table, or data structure, used to implement an
associative array, a structure than can map keys to values. In VES 6.0,
all name-value pair structures were changed to hash maps (i.e., {‘name’:
‘keyName’, ‘value’: ‘keyValue’} was replaced with {‘keyName’:
‘keyValue’}).

**ICE**. Incubation and Certification Environment. Facilitates vendors
and third-party in developing virtual network functions using ONAP and a
network cloud.

**IPMI**. The `Intelligent Platform Management
Interface <https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface>`__.

**JSON**. Java Script Object Notation. JSON is an
`open-standard <https://en.wikipedia.org/wiki/Open_standard>`__ `file
format <https://en.wikipedia.org/wiki/File_format>`__ that uses
`human-readable <https://en.wikipedia.org/wiki/Human-readable_medium>`__
text to transmit data objects consisting of `attribute–value
pairs <https://en.wikipedia.org/wiki/Attribute%E2%80%93value_pair>`__
and `array data types <https://en.wikipedia.org/wiki/Array_data_type>`__
(or any other
`serializable <https://en.wikipedia.org/wiki/Serialization>`__ value).
It is a very common `data <https://en.wikipedia.org/wiki/Data>`__ format
used for
`asynchronous <https://en.wikipedia.org/wiki/Asynchronous_I/O>`__
browser–server communication.

**NF**. Network Function. Generalized name for a VNF or PNF.

**NFC**. Network Function Component. Generalized name for a VNFC or a
component of a PNF.

**ONAP**. `Open Network Automation Platform <https://www.onap.org/>`__.

**PNF**. Physical Network Function.

**Policy**. Course of action for the management of the network. The ONAP
Policy Framework is a comprehensive policy design, deployment, and
execution environment. The Policy Framework is the **decision making**
component in `an ONAP
system <https://www.onap.org/wp-content/uploads/sites/20/2017/12/ONAP_CaseSolution_Architecture_120817_FNL.pdf>`__.
It allows you to specify, deploy, and execute the governance of the
features and functions in your ONAP system, be they closed loop,
orchestration, or more traditional open loop use case implementations.
The Policy Framework is the component that is the source of truth for
all policy decisions.

**Reporting Entity Name**. Name of the entity reporting the event or
detecting a problem in another vnf/vm or pnf which is experiencing the
problem. May be the same as the sourceName. Not used for performance
measurements currently.

**SDC**. Service Design and Creation Platform: The ONAP visual modeling
and design tool. It creates internal metadata that describes assets used
by all ONAP components, both at design time and run time. The SDC
manages the content of a catalog, and assemblies of selected catalog to
define how and when VNFs are realized in a target environment.  

**Source Name**: Name of the entity experiencing the event issue, which
may be detected and reported by a separate reporting entity. The
sourceName identifies the device for which data is collected. A valid
sourceName must be inventoried in A&AI.

**Specific Problem**. Description of the alarm or problem.

**VES**. Virtual Function Event Stream. In 6.0, the definition of VES
was expanded to include event streaming for VNF, PNF and infrastructure.
The VES Event Listener can receive any event sent in the VES Common
Event Format.

**VES Event Listener**. A RESTful connectionless push event listener
capable of receiving single events or batches of events sent in the
Common Event Format.

**VM**. Virtual Machine.

**VNF**. Virtual Network Function. A VNF is a virtualized task formerly
carried out by proprietary, dedicated network hardware. (Examples:
virtual firewall, virtual DNS). A VNF can also be defined as a specific
kind of Vendor Software Product.

**YAML**. A `data serialization
language <https://en.wikipedia.org/wiki/Data_serialization_language>`__
and superset of JSON.

**VNFC**. Virtual Network Function Component. A VNFC is a part of a VNF.
It is a stand-alone executable that is loosely-coupled, granular,
re-usable, and responsible for a single capability.

Appendix: Historical Change Log
===============================

For the latest changes, see the Change Block just before the Table of
Contents.

+-----------------------+-----------------------+-----------------------+
| Date                  | Revision              | Description           |
+-----------------------+-----------------------+-----------------------+
| 5/22/2015             | 0.1                   | Initial Release -     |
|                       |                       | Draft                 |
+-----------------------+-----------------------+-----------------------+
| 5/29/2015             | 0.2                   | -  Introduction:      |
|                       |                       |       removed all     |
|                       |                       |       system names    |
|                       |                       |       and references  |
|                       |                       |       to internal     |
|                       |                       |       AT&T components |
|                       |                       |                       |
|                       |                       | -  Security: changed  |
|                       |                       |       ‘event          |
|                       |                       |       publisher’ to   |
|                       |                       |       ‘event source’  |
|                       |                       |                       |
|                       |                       | -  Generic Event      |
|                       |                       |       Format: updated |
|                       |                       |       the JSON schema |
|                       |                       |       per the below:  |
|                       |                       |                       |
|                       |                       | -  eventHeader:       |
|                       |                       |       clarified the   |
|                       |                       |       description of  |
|                       |                       |       id, made        |
|                       |                       |       sourceId a      |
|                       |                       |       required field, |
|                       |                       |       changed the     |
|                       |                       |       datatype of     |
|                       |                       |       timestamps to   |
|                       |                       |       timestamp [ ]   |
|                       |                       |                       |
|                       |                       | -  performanceFields: |
|                       |                       |       removed         |
|                       |                       |       overflowFields  |
|                       |                       |                       |
|                       |                       | -  tmestamp: added a  |
|                       |                       |       description of  |
|                       |                       |       this datatype   |
|                       |                       |                       |
|                       |                       | -  Exceptions: fixed  |
|                       |                       |       indentation of  |
|                       |                       |       sections        |
|                       |                       |                       |
|                       |                       | -  Approvers: updated |
|                       |                       |       the list of     |
|                       |                       |       approvers and   |
|                       |                       |       added attuids   |
+-----------------------+-----------------------+-----------------------+
| 6/3/2015              | 0.3                   | -  Updated the        |
|                       |                       |       security        |
|                       |                       |       section to use  |
|                       |                       |       HTTP Basic      |
|                       |                       |       Authentication  |
|                       |                       |       per AT&T REST   |
|                       |                       |       standards.      |
|                       |                       |       Updated the     |
|                       |                       |       input           |
|                       |                       |       parameters and  |
|                       |                       |       messaging       |
|                       |                       |       examples to use |
|                       |                       |       the new         |
|                       |                       |       security        |
|                       |                       |       scheme.         |
+-----------------------+-----------------------+-----------------------+
| 6/5/2015              | 0.4                   | -  Added otherFields  |
|                       |                       |       sub section to  |
|                       |                       |       the defined     |
|                       |                       |       datatypes       |
|                       |                       |                       |
|                       |                       | -  Added locale field |
|                       |                       |       to the          |
|                       |                       |       eventHeader.    |
+-----------------------+-----------------------+-----------------------+
| 6/5/2015              | 0.5                   | -  Updated the        |
|                       |                       |       embedded event  |
|                       |                       |       format json     |
|                       |                       |       schema to match |
|                       |                       |       the changes     |
|                       |                       |       made in v0.4    |
+-----------------------+-----------------------+-----------------------+
| 6/10/2015             | 0.6                   | -  Updated the        |
|                       |                       |       {ServerRoot}    |
|                       |                       |       format to       |
|                       |                       |       contain an      |
|                       |                       |       optional        |
|                       |                       |       routing path    |
|                       |                       |       (for D2 service |
|                       |                       |       modules).       |
+-----------------------+-----------------------+-----------------------+
| 7/7/2015              | 0.7                   |    Common Event       |
|                       |                       |    Format updates:    |
|                       |                       |                       |
|                       |                       | -  EventHeader: added |
|                       |                       |       ‘measurement’   |
|                       |                       |       to the ‘domain’ |
|                       |                       |       enumeration;    |
|                       |                       |       changed         |
|                       |                       |       ‘locale’ to     |
|                       |                       |       ‘location’ and  |
|                       |                       |       clarified in    |
|                       |                       |       the description |
|                       |                       |       that this       |
|                       |                       |       should be a     |
|                       |                       |       clli code       |
|                       |                       |                       |
|                       |                       | -  Added a            |
|                       |                       |       MeasurementFiel |
|                       |                       | ds                    |
|                       |                       |       datatype, which |
|                       |                       |       required the    |
|                       |                       |       addition of the |
|                       |                       |       following       |
|                       |                       |       datatypes:      |
|                       |                       |       codecsInUse,    |
|                       |                       |       cpuUsage,       |
|                       |                       |       diskUsage,      |
|                       |                       |       featuresInUse,  |
|                       |                       |       memoryUsage     |
+-----------------------+-----------------------+-----------------------+
| 7/15/2015             | 1.0                   | -  Changed            |
|                       |                       |    sourceInstance in  |
|                       |                       |    the eventHeader to |
|                       |                       |    be an array of     |
|                       |                       |    name value pairs   |
|                       |                       |                       |
|                       |                       | -  Changed the        |
|                       |                       |    performanceFields  |
|                       |                       |    block to           |
|                       |                       |    thresholdCrossingA |
|                       |                       | lertFields.           |
|                       |                       |    Updated the domain |
|                       |                       |    field of the       |
|                       |                       |    eventHeader to     |
|                       |                       |    match.             |
+-----------------------+-----------------------+-----------------------+
| 7/23/2015             | v1.1                  |    Changes to         |
|                       |                       |    eventHeader data   |
|                       |                       |    format:            |
|                       |                       |                       |
|                       |                       | -  moved              |
|                       |                       |       sourceInstance  |
|                       |                       |       to              |
|                       |                       |       internalHeaderF |
|                       |                       | ields                 |
|                       |                       |                       |
|                       |                       | -  moved              |
|                       |                       |       serviceInstance |
|                       |                       | Id                    |
|                       |                       |       to              |
|                       |                       |       internalHeaderF |
|                       |                       | ields                 |
|                       |                       |                       |
|                       |                       | -  moved productId to |
|                       |                       |       internalHeaderF |
|                       |                       | ields                 |
|                       |                       |                       |
|                       |                       | -  moved subscriberId |
|                       |                       |       to              |
|                       |                       |       internalHeaderF |
|                       |                       | ields                 |
|                       |                       |                       |
|                       |                       | -  moved location to  |
|                       |                       |       internalHeaderF |
|                       |                       | ields                 |
|                       |                       |                       |
|                       |                       | -  added the          |
|                       |                       |       following new   |
|                       |                       |       fields in       |
|                       |                       |       internalHeaderF |
|                       |                       | ields:                |
|                       |                       |       policyType,     |
|                       |                       |       policyName,     |
|                       |                       |       correlationEven |
|                       |                       | tType,                |
|                       |                       |       correlationType |
|                       |                       | ,                     |
|                       |                       |       correlationName |
|                       |                       | ,                     |
|                       |                       |       correlationRoot |
|                       |                       | EventId               |
|                       |                       |                       |
|                       |                       | -  Changes to         |
|                       |                       |       faultFields     |
|                       |                       |       data format:    |
|                       |                       |                       |
|                       |                       | -  moved the          |
|                       |                       |       eventSourceDevi |
|                       |                       | ceDescription         |
|                       |                       |       to              |
|                       |                       |       internalFaultFi |
|                       |                       | elds                  |
|                       |                       |       and renamed it  |
|                       |                       |       equipmentVendor |
|                       |                       | Model                 |
|                       |                       |                       |
|                       |                       | -  moved              |
|                       |                       |       eventSourceHost |
|                       |                       | name                  |
|                       |                       |       to              |
|                       |                       |       internalFaultFi |
|                       |                       | elds                  |
|                       |                       |                       |
|                       |                       | -  changed            |
|                       |                       |       alarmObjectInte |
|                       |                       | rface                 |
|                       |                       |       to              |
|                       |                       |       alarmInterfaceA |
|                       |                       |                       |
|                       |                       | -  changed            |
|                       |                       |       alarmRemoteObje |
|                       |                       | ct                    |
|                       |                       |       to              |
|                       |                       |       alarmRemoteObje |
|                       |                       | ctZ                   |
|                       |                       |       and moved it to |
|                       |                       |       internalFaultFi |
|                       |                       | elds                  |
|                       |                       |                       |
|                       |                       | -  changed            |
|                       |                       |       alarmRemoteObje |
|                       |                       | ctInterface           |
|                       |                       |       to              |
|                       |                       |       alarmInterfaceZ |
|                       |                       |       and moved it to |
|                       |                       |       internalFaultFi |
|                       |                       | elds                  |
|                       |                       |                       |
|                       |                       | -  Changes to         |
|                       |                       |       thresholdCrossi |
|                       |                       | ngFields              |
|                       |                       |       data format:    |
|                       |                       |                       |
|                       |                       | -  changed several    |
|                       |                       |       references from |
|                       |                       |       the old         |
|                       |                       |       ‘performanceFie |
|                       |                       | lds’                  |
|                       |                       |       block to the    |
|                       |                       |       new             |
|                       |                       |       ‘thresholdCross |
|                       |                       | ingFields’            |
|                       |                       |       block           |
|                       |                       |                       |
|                       |                       | -  Other:             |
|                       |                       |                       |
|                       |                       | -  Fixed several      |
|                       |                       |       comma and colon |
|                       |                       |       syntax errors   |
|                       |                       |       in the JSON     |
|                       |                       |       schema as       |
|                       |                       |       detected by a   |
|                       |                       |       JSON schema     |
|                       |                       |       syntax checker. |
+-----------------------+-----------------------+-----------------------+
| 8/11/2015             | v1.2                  |    Timestamp format:  |
|                       |                       |                       |
|                       |                       | -  Section 4.18:      |
|                       |                       |       added a note in |
|                       |                       |       the datetime    |
|                       |                       |       field of the    |
|                       |                       |       Timestamp       |
|                       |                       |       datatype        |
|                       |                       |       specifying the  |
|                       |                       |       (GMT) format    |
|                       |                       |       required        |
|                       |                       |                       |
|                       |                       | -  Updated the JSON   |
|                       |                       |       schema with the |
|                       |                       |       same            |
|                       |                       |       information     |
|                       |                       |                       |
|                       |                       | ..                    |
|                       |                       |                       |
|                       |                       |    Event Header       |
|                       |                       |    Severity           |
|                       |                       |    Enumeration:       |
|                       |                       |                       |
|                       |                       | -  Section 4.8:       |
|                       |                       |       modified the    |
|                       |                       |       severity        |
|                       |                       |       enumeration to  |
|                       |                       |       remove the      |
|                       |                       |       numbers in      |
|                       |                       |       parentheses     |
|                       |                       |       that followed   |
|                       |                       |       the names. The  |
|                       |                       |       names were not  |
|                       |                       |       changed.        |
|                       |                       |                       |
|                       |                       | -  Updated the JSON   |
|                       |                       |       schema with the |
|                       |                       |       same            |
|                       |                       |       information.    |
+-----------------------+-----------------------+-----------------------+
| 8/20/2015             | v1.3                  | JSON Schema rev’d to  |
|                       |                       | v9:                   |
|                       |                       |                       |
|                       |                       | -  Alphabetized all   |
|                       |                       |       fields in the   |
|                       |                       |       JSON schema     |
|                       |                       |                       |
|                       |                       | -  Fixed the way      |
|                       |                       |       arrays were     |
|                       |                       |       specified (JSON |
|                       |                       |       schema syntax   |
|                       |                       |       issue)          |
|                       |                       |                       |
|                       |                       | Sample Responses:     |
|                       |                       |                       |
|                       |                       | -  2.1.1.1:           |
|                       |                       |       alphabetized    |
|                       |                       |       fields, fixed   |
|                       |                       |       timestamps      |
|                       |                       |       array           |
|                       |                       |       depiction,      |
|                       |                       |       fixed severity  |
|                       |                       |       enum value to   |
|                       |                       |       conform to      |
|                       |                       |       latest format   |
|                       |                       |                       |
|                       |                       | -  6.2.6.1:           |
|                       |                       |       alphabetized    |
|                       |                       |       fields, fixed   |
|                       |                       |       timestamps      |
|                       |                       |       array           |
|                       |                       |       depiction,      |
|                       |                       |       fixed severity  |
|                       |                       |       enum value to   |
|                       |                       |       conform to      |
|                       |                       |       latest format   |
|                       |                       |                       |
|                       |                       | -  6.3.6.1:           |
|                       |                       |       alphabetized    |
|                       |                       |       fields, fixed   |
|                       |                       |       timestamps      |
|                       |                       |       array           |
|                       |                       |       depiction,      |
|                       |                       |       fixed severity  |
|                       |                       |       enum value to   |
|                       |                       |       conform to      |
|                       |                       |       latest format   |
|                       |                       |                       |
|                       |                       | -  6.4.6.1:           |
|                       |                       |       alphabetized    |
|                       |                       |       fields, fixed   |
|                       |                       |       timestamps      |
|                       |                       |       array           |
|                       |                       |       depiction,      |
|                       |                       |       fixed eventList |
|                       |                       |       array           |
|                       |                       |       depection,      |
|                       |                       |       fixed severity  |
|                       |                       |       enum value to   |
|                       |                       |       conform to      |
|                       |                       |       latest format   |
+-----------------------+-----------------------+-----------------------+
| 9/16/2015             | v1.4                  | JSON Schema rev’d to  |
|                       |                       | v10:                  |
|                       |                       |                       |
|                       |                       | -  Fixed an error in  |
|                       |                       |       the way that    |
|                       |                       |       the top level   |
|                       |                       |       “event” object  |
|                       |                       |       was specified   |
|                       |                       |       in the v9 json  |
|                       |                       |       schema. This    |
|                       |                       |       was discovered  |
|                       |                       |       when validating |
|                       |                       |       examples        |
|                       |                       |       against the     |
|                       |                       |       schema using    |
|                       |                       |       this site:      |
|                       |                       |       http://json-sch |
|                       |                       | ema-validator.herokua |
|                       |                       | pp.com/index.jsp.     |
|                       |                       |                       |
|                       |                       | -  Changed the        |
|                       |                       |       embedded json   |
|                       |                       |       file in section |
|                       |                       |       4               |
|                       |                       |                       |
|                       |                       | Sample Responses:     |
|                       |                       |                       |
|                       |                       | -  Removed an extra   |
|                       |                       |       comma after the |
|                       |                       |       timestamp brace |
|                       |                       |       in section      |
|                       |                       |       6.2.6 and       |
|                       |                       |       6.3.6.          |
+-----------------------+-----------------------+-----------------------+
| 11/11/2015            | v1.5                  | Section 4 was the     |
|                       |                       | only section changed: |
|                       |                       | JSON Schema rev’d to  |
|                       |                       | v11 and Datatype      |
|                       |                       | tables were updated   |
|                       |                       | to match. Numerous    |
|                       |                       | data structure        |
|                       |                       | changes were made     |
|                       |                       | based on VNF vendor   |
|                       |                       | proof of concept      |
|                       |                       | feedback. Modified    |
|                       |                       | sample requests and   |
|                       |                       | responses to match.   |
+-----------------------+-----------------------+-----------------------+
| 11/12/2015            | v1.6                  | -  The                |
|                       |                       |       internalFaultFi |
|                       |                       | elds                  |
|                       |                       |       were merged     |
|                       |                       |       into the        |
|                       |                       |       internalHeaderF |
|                       |                       | ields;                |
|                       |                       |       then the        |
|                       |                       |       internalFaultFi |
|                       |                       | elds                  |
|                       |                       |       datatype was    |
|                       |                       |       deleted.        |
|                       |                       |                       |
|                       |                       | -  Updated the JSON   |
|                       |                       |       schema to v12.  |
|                       |                       |                       |
|                       |                       | -  Also corrected     |
|                       |                       |       some background |
|                       |                       |       color issues in |
|                       |                       |       the sample      |
|                       |                       |       requests and    |
|                       |                       |       responses.      |
+-----------------------+-----------------------+-----------------------+
| 1/18/2016             | v1.7                  | -  Section 2 changes: |
|                       |                       |       updated the     |
|                       |                       |       sample request  |
|                       |                       |       to conform with |
|                       |                       |       the changes     |
|                       |                       |       below           |
|                       |                       |                       |
|                       |                       | -  Section 4 datatype |
|                       |                       |       changes:        |
|                       |                       |                       |
|                       |                       | -  Changed            |
|                       |                       |       'eventHeader'   |
|                       |                       |       to              |
|                       |                       |       'commonEventHea |
|                       |                       | der'                  |
|                       |                       |                       |
|                       |                       | -  Moved              |
|                       |                       |       'eventSeverity' |
|                       |                       |       from the        |
|                       |                       |       'commonEventHea |
|                       |                       | der'                  |
|                       |                       |       to              |
|                       |                       |       'faultFields'   |
|                       |                       |                       |
|                       |                       | -  Added 'priority'   |
|                       |                       |       to              |
|                       |                       |       'commonEventHea |
|                       |                       | der'                  |
|                       |                       |                       |
|                       |                       | -  moved 'vFstatus'   |
|                       |                       |       to              |
|                       |                       |       'faultFields'   |
|                       |                       |                       |
|                       |                       | -  removed            |
|                       |                       |       'firstDateTime' |
|                       |                       |       and             |
|                       |                       |       'lastDateTime'  |
|                       |                       |       and changed     |
|                       |                       |       'firstEpoch' to |
|                       |                       |       'startEpochMicr |
|                       |                       | osec'                 |
|                       |                       |       and changed     |
|                       |                       |       'lastEpoch' to  |
|                       |                       |       'lastEpochMicro |
|                       |                       | sec'.                 |
|                       |                       |                       |
|                       |                       | -  Added              |
|                       |                       |       'functionalRole |
|                       |                       | '                     |
|                       |                       |       to the          |
|                       |                       |       commonEventHead |
|                       |                       | er                    |
|                       |                       |                       |
|                       |                       | -  In the             |
|                       |                       |       commonEventHead |
|                       |                       | er,                   |
|                       |                       |       changed the     |
|                       |                       |       'eventDomain'   |
|                       |                       |       enumeration to  |
|                       |                       |       remove          |
|                       |                       |       'measurements'  |
|                       |                       |       and add         |
|                       |                       |       'measurementsFo |
|                       |                       | rVfScaling'.          |
|                       |                       |                       |
|                       |                       | -  Changed the        |
|                       |                       |       'measurementFie |
|                       |                       | lds'                  |
|                       |                       |       to              |
|                       |                       |       'measurementsFo |
|                       |                       | rVfScalingFields'     |
|                       |                       |                       |
|                       |                       | -  In the             |
|                       |                       |       commonEventHead |
|                       |                       | er,                   |
|                       |                       |       changed the     |
|                       |                       |       following       |
|                       |                       |       fields:         |
|                       |                       |                       |
|                       |                       | -  'eventDomain' to   |
|                       |                       |       'domain'        |
|                       |                       |                       |
|                       |                       | -  'eventSequence' to |
|                       |                       |       'sequence'      |
|                       |                       |                       |
|                       |                       | -  'eventSourceId' to |
|                       |                       |       'sourceId'      |
|                       |                       |                       |
|                       |                       | -  'eventSounceName'  |
|                       |                       |       to 'sourceName' |
|                       |                       |                       |
|                       |                       | -  Updated the JSON   |
|                       |                       |       schema to v13   |
|                       |                       |                       |
|                       |                       | -  Section 6 changes: |
|                       |                       |       updated the     |
|                       |                       |       input           |
|                       |                       |       parameters and  |
|                       |                       |       sample requests |
|                       |                       |       to conform to   |
|                       |                       |       the changes     |
|                       |                       |       above.          |
|                       |                       |                       |
|                       |                       | -  Section 7: changed |
|                       |                       |       the section     |
|                       |                       |       from Approvers  |
|                       |                       |       to              |
|                       |                       |       Contributors.   |
+-----------------------+-----------------------+-----------------------+
| 1/22/2016             | v1.8                  | -  Section 4: Added   |
|                       |                       |       support for     |
|                       |                       |       ‘mobileFlow’ in |
|                       |                       |       the             |
|                       |                       |       commonEventHead |
|                       |                       | er                    |
|                       |                       |       ‘domain’        |
|                       |                       |       enumeration.    |
|                       |                       |       Added the       |
|                       |                       |       mobileFlowField |
|                       |                       | s                     |
|                       |                       |       datatype and    |
|                       |                       |       the             |
|                       |                       |       gtpPerFlowMetri |
|                       |                       | cs                    |
|                       |                       |       datatype        |
|                       |                       |       referenced by   |
|                       |                       |       that datatype.  |
|                       |                       |                       |
|                       |                       | -  Section 7:         |
|                       |                       |       alphabetized    |
|                       |                       |       the             |
|                       |                       |       contributors    |
+-----------------------+-----------------------+-----------------------+
| 2/11/2016             | v1.9                  | -  Added section 1.3: |
|                       |                       |       Naming Standard |
|                       |                       |       for Event Types |
+-----------------------+-----------------------+-----------------------+
| 2/12/2016             | v2.0                  | -  Updated request –  |
|                       |                       |       response        |
|                       |                       |       examples to     |
|                       |                       |       reflect the     |
|                       |                       |       naming          |
|                       |                       |       standards for   |
|                       |                       |       event types     |
|                       |                       |       introduced in   |
|                       |                       |       v1.9.           |
|                       |                       |                       |
|                       |                       | -  Added a paragraph  |
|                       |                       |       on use of Avro  |
|                       |                       |       as a transport  |
|                       |                       |       in section 1.4  |
+-----------------------+-----------------------+-----------------------+
| 3/11/2016             | v2.1                  | -  Updated the        |
|                       |                       |       embedded JSON   |
|                       |                       |       schema to v15   |
|                       |                       |       to fix a typo   |
|                       |                       |       in the required |
|                       |                       |       fields for the  |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields,      |
|                       |                       |       namely, changed |
|                       |                       |       ‘configuredEnti |
|                       |                       | tes’                  |
|                       |                       |       to              |
|                       |                       |       ‘configuredEnti |
|                       |                       | ties’.                |
|                       |                       |       Additionally,   |
|                       |                       |       added an ‘Event |
|                       |                       |       Listener’ title |
|                       |                       |       block at the    |
|                       |                       |       bottom of the   |
|                       |                       |       file with a     |
|                       |                       |       single required |
|                       |                       |       event object.   |
+-----------------------+-----------------------+-----------------------+
| 3/15/2016             | v2.2                  | -  Added              |
|                       |                       |       mobileFlowField |
|                       |                       | s                     |
|                       |                       |       to the event    |
|                       |                       |       datatype        |
|                       |                       |       definition in   |
|                       |                       |       section 4.7 and |
|                       |                       |       updated the     |
|                       |                       |       embedded json   |
|                       |                       |       schema at the   |
|                       |                       |       top of section  |
|                       |                       |       4 to v16.       |
+-----------------------+-----------------------+-----------------------+
| 4/26/2016             | v2.3                  | -  Generic Event      |
|                       |                       |       Format updates: |
|                       |                       |       1) made         |
|                       |                       |       ‘priority’      |
|                       |                       |       lowercase in    |
|                       |                       |       the Word doc    |
|                       |                       |       table for       |
|                       |                       |       commonEventHead |
|                       |                       | er;                   |
|                       |                       |       2) added        |
|                       |                       |       ‘requestError’  |
|                       |                       |       data structure  |
|                       |                       |       to the Word doc |
|                       |                       |       and JSON schema |
|                       |                       |       (which is now   |
|                       |                       |       at v17)         |
+-----------------------+-----------------------+-----------------------+
| 4/27/2016             | v2.4                  | -  JSON Schema: In    |
|                       |                       |       the 'event'     |
|                       |                       |       data structure, |
|                       |                       |       changed         |
|                       |                       |       'thresholdCross |
|                       |                       | ingFields'            |
|                       |                       |       to              |
|                       |                       |       'thresholdCross |
|                       |                       | ingAlertFields'       |
|                       |                       |       to product v18  |
|                       |                       |       of the schema.  |
|                       |                       |                       |
|                       |                       | -  'codecsInUse' data |
|                       |                       |       structure:      |
|                       |                       |       changed         |
|                       |                       |       'numberInUse'   |
|                       |                       |       to              |
|                       |                       |       'codecUtilizati |
|                       |                       | on’                   |
+-----------------------+-----------------------+-----------------------+
| 5/26/2016             | v2.5                  | -  Changed responses  |
|                       |                       |       from ‘204 No    |
|                       |                       |       Content’ to     |
|                       |                       |       ‘202 Accepted’  |
|                       |                       |       and added a     |
|                       |                       |       body to the     |
|                       |                       |       response that   |
|                       |                       |       enable AT&T to  |
|                       |                       |       throttle the    |
|                       |                       |       events being    |
|                       |                       |       sent and/or to  |
|                       |                       |       request the     |
|                       |                       |       current state   |
|                       |                       |       of throttling   |
|                       |                       |       at the event    |
|                       |                       |       source.         |
|                       |                       |                       |
|                       |                       | -  Added new          |
|                       |                       |       datatypes to    |
|                       |                       |       support the     |
|                       |                       |       above:          |
|                       |                       |       eventDomainThro |
|                       |                       | ttleSpecification,    |
|                       |                       |       eventDomainThro |
|                       |                       | ttleSpecificationList |
|                       |                       | ,                     |
|                       |                       |       eventThrottling |
|                       |                       | State,                |
|                       |                       |       suppressedNvPai |
|                       |                       | rs                    |
|                       |                       |                       |
|                       |                       | -  Modifed the        |
|                       |                       |       commonEventForm |
|                       |                       | at                    |
|                       |                       |       json schema to  |
|                       |                       |       v19             |
|                       |                       |                       |
|                       |                       | -  Note: for the      |
|                       |                       |       VendorEventList |
|                       |                       | ener:                 |
|                       |                       |       added new       |
|                       |                       |       licensing       |
|                       |                       |       language on the |
|                       |                       |       back of the     |
|                       |                       |       title page;     |
|                       |                       |       added an        |
|                       |                       |       “attCopyrightNo |
|                       |                       | tice”                 |
|                       |                       |       definition at   |
|                       |                       |       the top of the  |
|                       |                       |       commonEventForm |
|                       |                       | at_Vendors.json       |
|                       |                       |       file; also      |
|                       |                       |       removed all     |
|                       |                       |       references to   |
|                       |                       |       internalHeaderF |
|                       |                       | ields                 |
|                       |                       |       from this file  |
|                       |                       |       and from the    |
|                       |                       |       VendorEventList |
|                       |                       | ener                  |
|                       |                       |       spec.           |
+-----------------------+-----------------------+-----------------------+
| 8/9/2016              | v2.6                  | -  commonHeader:      |
|                       |                       |       added a note on |
|                       |                       |       the description |
|                       |                       |       of sourceId and |
|                       |                       |       sourceName in   |
|                       |                       |       the             |
|                       |                       |       commonHeader:   |
|                       |                       |       "use            |
|                       |                       |       reportingEntity |
|                       |                       |       for domains     |
|                       |                       |       that provide    |
|                       |                       |       more detailed   |
|                       |                       |       source info"    |
|                       |                       |                       |
|                       |                       | -  commonHeader:      |
|                       |                       |       deleted the     |
|                       |                       |       capacity,       |
|                       |                       |       measurementsFor |
|                       |                       | VfScaling             |
|                       |                       |       and usage       |
|                       |                       |       domains in the  |
|                       |                       |       domain          |
|                       |                       |       enumeration     |
|                       |                       |                       |
|                       |                       | -  commonHeader:      |
|                       |                       |       added the       |
|                       |                       |       following       |
|                       |                       |       domains to the  |
|                       |                       |       domain          |
|                       |                       |       enumeration:    |
|                       |                       |       licensingKci,   |
|                       |                       |       scalingKpi,     |
|                       |                       |       stateChange     |
|                       |                       |                       |
|                       |                       | -  event: removed     |
|                       |                       |       references to   |
|                       |                       |       capacityFields, |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields       |
|                       |                       |       and usageFields |
|                       |                       |       and added       |
|                       |                       |       references to   |
|                       |                       |       licensingKciFie |
|                       |                       | lds,                  |
|                       |                       |       scalingKpiField |
|                       |                       | s,                    |
|                       |                       |       stateChangeFiel |
|                       |                       | ds                    |
|                       |                       |                       |
|                       |                       | -  licensingKciFields |
|                       |                       | :                     |
|                       |                       |       added this      |
|                       |                       |       section along   |
|                       |                       |       with            |
|                       |                       |       'additionalMeas |
|                       |                       | urements',            |
|                       |                       |       which is an     |
|                       |                       |       optional list   |
|                       |                       |       of              |
|                       |                       |       measurementGrou |
|                       |                       | p                     |
|                       |                       |       structures.     |
|                       |                       |       Changed the     |
|                       |                       |       name of         |
|                       |                       |       kciFieldsVersio |
|                       |                       | n                     |
|                       |                       |       to              |
|                       |                       |       licensingKciFie |
|                       |                       | ldsVersion.           |
|                       |                       |                       |
|                       |                       | -  scalingKpiFields:  |
|                       |                       |       added this      |
|                       |                       |       section but     |
|                       |                       |       changed         |
|                       |                       |       measurementFiel |
|                       |                       | dsVersion             |
|                       |                       |       to              |
|                       |                       |       scalingKpiField |
|                       |                       | sVersion              |
|                       |                       |                       |
|                       |                       | -  stateChangeFields: |
|                       |                       |       added this      |
|                       |                       |       section along   |
|                       |                       |       with            |
|                       |                       |       'additionalFiel |
|                       |                       | ds',                  |
|                       |                       |       which is an     |
|                       |                       |       optional list   |
|                       |                       |       of name-value   |
|                       |                       |       pairs. Other    |
|                       |                       |       fields included |
|                       |                       |       newState and    |
|                       |                       |       oldState which  |
|                       |                       |       were            |
|                       |                       |       enumerations of |
|                       |                       |       the following   |
|                       |                       |       possible        |
|                       |                       |       states:         |
|                       |                       |       'inService',    |
|                       |                       |       'maintenance',  |
|                       |                       |       'outOfService'  |
|                       |                       |                       |
|                       |                       | -  sysLogFields:      |
|                       |                       |       added           |
|                       |                       |       'additionalFiel |
|                       |                       | ds',                  |
|                       |                       |       which is an     |
|                       |                       |       optional list   |
|                       |                       |       of name-value   |
|                       |                       |       pairs           |
|                       |                       |                       |
|                       |                       | -  vNicUsage: added   |
|                       |                       |       two required    |
|                       |                       |       fields to the   |
|                       |                       |       vNicUsage data  |
|                       |                       |       structure:      |
|                       |                       |       packetsIn and   |
|                       |                       |       packetsOut      |
+-----------------------+-----------------------+-----------------------+
| 8/10/2016             | v2.7                  | -  commonHeader:      |
|                       |                       |       removed the     |
|                       |                       |       note on the     |
|                       |                       |       description of  |
|                       |                       |       sourceId and    |
|                       |                       |       sourceName in   |
|                       |                       |       the             |
|                       |                       |       commonHeader:   |
|                       |                       |       "use            |
|                       |                       |       reportingEntity |
|                       |                       |       for domains     |
|                       |                       |       that provide    |
|                       |                       |       more detailed   |
|                       |                       |       source info"    |
|                       |                       |                       |
|                       |                       | -  commonHeader:      |
|                       |                       |       added           |
|                       |                       |       measurementsFor |
|                       |                       | VfScaling             |
|                       |                       |       domain back and |
|                       |                       |       removed the     |
|                       |                       |       licensingKci    |
|                       |                       |       and scalingKpi  |
|                       |                       |       domains         |
|                       |                       |                       |
|                       |                       | -  event: removed     |
|                       |                       |       references to   |
|                       |                       |       licensingKciFie |
|                       |                       | lds                   |
|                       |                       |       and             |
|                       |                       |       scalingKpiField |
|                       |                       | s;                    |
|                       |                       |       added           |
|                       |                       |       references to   |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields       |
|                       |                       |                       |
|                       |                       | -  measurementsForVfS |
|                       |                       | calingFields:         |
|                       |                       |       combined the    |
|                       |                       |       kciDetail and   |
|                       |                       |       kpiDetail       |
|                       |                       |       structures into |
|                       |                       |       the             |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields       |
|                       |                       |       structure;      |
|                       |                       |       referenced the  |
|                       |                       |       errors          |
|                       |                       |       structure       |
|                       |                       |                       |
|                       |                       | -  errors: added a    |
|                       |                       |       new structure   |
|                       |                       |       to capture the  |
|                       |                       |       receive and     |
|                       |                       |       transmit errors |
|                       |                       |       for the         |
|                       |                       |       measurements    |
|                       |                       |       domain          |
|                       |                       |                       |
|                       |                       | -  removed the        |
|                       |                       |       following       |
|                       |                       |       structures:     |
|                       |                       |       kci, kpi,       |
|                       |                       |       scalingKpiField |
|                       |                       | s                     |
|                       |                       |       and             |
|                       |                       |       licensingKciFie |
|                       |                       | lds                   |
|                       |                       |                       |
|                       |                       | -  eventDomainThrottl |
|                       |                       | eSpecification:       |
|                       |                       |       updated the     |
|                       |                       |       reference to    |
|                       |                       |       commonEventHead |
|                       |                       | er                    |
|                       |                       |       domain field    |
|                       |                       |                       |
|                       |                       | -  faultFields:       |
|                       |                       |       removed the     |
|                       |                       |       numbers from    |
|                       |                       |       the enumerated  |
|                       |                       |       strings for     |
|                       |                       |       eventSourceType |
|                       |                       |                       |
|                       |                       | -  vNicUsage: made    |
|                       |                       |       the broadcast,  |
|                       |                       |       multicast and   |
|                       |                       |       unicast fields  |
|                       |                       |       optional        |
|                       |                       |                       |
|                       |                       | -  contributors:      |
|                       |                       |       updated Alok’s  |
|                       |                       |       organizational  |
|                       |                       |       area            |
+-----------------------+-----------------------+-----------------------+
| 8/12/2016             | v2.8                  | -  commonHeader:      |
|                       |                       |       copied the      |
|                       |                       |       descriptions of |
|                       |                       |       sourceId and    |
|                       |                       |       sourceName from |
|                       |                       |       the JSON schema |
|                       |                       |       into the word   |
|                       |                       |       document        |
|                       |                       |       tables.         |
|                       |                       |                       |
|                       |                       | -  sample request     |
|                       |                       |       examples: moved |
|                       |                       |       the             |
|                       |                       |       reportingEntity |
|                       |                       | Id                    |
|                       |                       |       and             |
|                       |                       |       reportingEntity |
|                       |                       | Names                 |
|                       |                       |       to the same     |
|                       |                       |       relative place  |
|                       |                       |       in all sample   |
|                       |                       |       requests in the |
|                       |                       |       document        |
|                       |                       |                       |
|                       |                       | -  Fixed the sample   |
|                       |                       |       request shown   |
|                       |                       |       for             |
|                       |                       |       publishEventBat |
|                       |                       | ch                    |
|                       |                       |       to take an      |
|                       |                       |       eventList as    |
|                       |                       |       input.          |
|                       |                       |                       |
|                       |                       | -  Fixed the sample   |
|                       |                       |       request shown   |
|                       |                       |       for             |
|                       |                       |       publishSpecific |
|                       |                       | Topic                 |
|                       |                       |       to put the      |
|                       |                       |       topic in the    |
|                       |                       |       URL             |
|                       |                       |                       |
|                       |                       | -  errors: changed    |
|                       |                       |       the             |
|                       |                       |       receiveErrors   |
|                       |                       |       and             |
|                       |                       |       transmitErrors  |
|                       |                       |       fields to be    |
|                       |                       |       datatype number |
|                       |                       |                       |
|                       |                       | -  codesInUse:        |
|                       |                       |       changed         |
|                       |                       |       'codecUtilizati |
|                       |                       | on'                   |
|                       |                       |       to              |
|                       |                       |       'numberinUse'   |
|                       |                       |                       |
|                       |                       | -  vNicUsage: updated |
|                       |                       |       the description |
|                       |                       |       of the fields   |
+-----------------------+-----------------------+-----------------------+
| 8/27/2016             | v2.9                  | -  Added a note       |
|                       |                       |       "(currently:    |
|                       |                       |       1.1)" in the    |
|                       |                       |       descriptions of |
|                       |                       |       the following   |
|                       |                       |       fields:         |
|                       |                       |       commonEventHead |
|                       |                       | er:version,           |
|                       |                       |       faultFields:fau |
|                       |                       | ltFieldsVersion,      |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields:measu |
|                       |                       | rementsForVfScalingFi |
|                       |                       | eldsVersion,          |
|                       |                       |       stateChangeFiel |
|                       |                       | ds:stateChangeFieldsV |
|                       |                       | ersion,               |
|                       |                       |       sysLogFields:sy |
|                       |                       | slogFieldsVersion,    |
|                       |                       |       thresholdCrossi |
|                       |                       | ngAlertFields:thresho |
|                       |                       | ldCrossingFieldsVersi |
|                       |                       | on                    |
|                       |                       |                       |
|                       |                       | -  stateChangeFields: |
|                       |                       |       made            |
|                       |                       |       stateInterface  |
|                       |                       |       mandatory       |
|                       |                       |                       |
|                       |                       | -  changed 'enum' to  |
|                       |                       |       'enumeration'   |
|                       |                       |       throughout      |
|                       |                       |       section 4 of    |
|                       |                       |       the document    |
|                       |                       |       (note: this     |
|                       |                       |       can't be done   |
|                       |                       |       in the JSON     |
|                       |                       |       schema).        |
|                       |                       |                       |
|                       |                       | -  measurementsForVfS |
|                       |                       | calingFields:         |
|                       |                       |       made the        |
|                       |                       |       following       |
|                       |                       |       fields          |
|                       |                       |       optional:       |
|                       |                       |       conurrentSessio |
|                       |                       | ns,                   |
|                       |                       |       configuredEntit |
|                       |                       | ites,                 |
|                       |                       |       cpuUsageArray,  |
|                       |                       |       fileSystemUsage |
|                       |                       | Array,                |
|                       |                       |       memoryConfigure |
|                       |                       | d,                    |
|                       |                       |       memoryUsed,     |
|                       |                       |       requestRate,    |
|                       |                       |       vNicUsageArray  |
|                       |                       |                       |
|                       |                       | -  measurementsForVfS |
|                       |                       | calingFields:         |
|                       |                       |       concurrentSessi |
|                       |                       | ons                   |
|                       |                       |       and             |
|                       |                       |       configuredEntit |
|                       |                       | ies:                  |
|                       |                       |       changed the     |
|                       |                       |       description to  |
|                       |                       |       support both    |
|                       |                       |       VMs and VNFs    |
|                       |                       |                       |
|                       |                       | -  measurementsFor    |
|                       |                       |       VfScalingFields |
|                       |                       | :                     |
|                       |                       |       clarified the   |
|                       |                       |       descriptions of |
|                       |                       |       latencyDistribu |
|                       |                       | tion,                 |
|                       |                       |       measurementInve |
|                       |                       | rval                  |
|                       |                       |       and requestRate |
|                       |                       |                       |
|                       |                       | -  syslogFields:      |
|                       |                       |       clarified the   |
|                       |                       |       descriptions of |
|                       |                       |       syslogSData,    |
|                       |                       |       syslogTag,      |
|                       |                       |       syslogVer       |
|                       |                       |                       |
|                       |                       | -  thresholdCrossingA |
|                       |                       | lertFields:           |
|                       |                       |       made the        |
|                       |                       |       following       |
|                       |                       |       fields optional |
|                       |                       |       and clarified   |
|                       |                       |       their           |
|                       |                       |       descriptions:   |
|                       |                       |       elementType,    |
|                       |                       |       networkService  |
|                       |                       |                       |
|                       |                       | -  command and        |
|                       |                       |       commandList:    |
|                       |                       |       created a list  |
|                       |                       |       of command      |
|                       |                       |       structures to   |
|                       |                       |       enable the      |
|                       |                       |       event collector |
|                       |                       |       to request      |
|                       |                       |       changes of      |
|                       |                       |       event sources.  |
|                       |                       |       Commands        |
|                       |                       |       consist of a    |
|                       |                       |       commandType     |
|                       |                       |       along with      |
|                       |                       |       optional fields |
|                       |                       |       (whose presence |
|                       |                       |       is indicated by |
|                       |                       |       the             |
|                       |                       |       commandType).   |
|                       |                       |       Three command   |
|                       |                       |       types are       |
|                       |                       |       currently       |
|                       |                       |       supported:      |
|                       |                       |       'measurementInt |
|                       |                       | evalChange',          |
|                       |                       |       ‘provideThrottl |
|                       |                       | ingState’             |
|                       |                       |       and             |
|                       |                       |       'throttlingSpec |
|                       |                       | ification'.           |
|                       |                       |                       |
|                       |                       | -  eventDomainThrottl |
|                       |                       | eSpecificationList:   |
|                       |                       |       removed this    |
|                       |                       |       and replaced it |
|                       |                       |       with            |
|                       |                       |       commandList.    |
|                       |                       |                       |
|                       |                       | -  Operations and     |
|                       |                       |       Sample          |
|                       |                       |       Requests:       |
|                       |                       |       modified the    |
|                       |                       |       operations and  |
|                       |                       |       samples to      |
|                       |                       |       support the new |
|                       |                       |       command and     |
|                       |                       |       commandList     |
|                       |                       |       structures.     |
+-----------------------+-----------------------+-----------------------+
| 9/1/2016              | v2.10                 | -  measurementsForVfS |
|                       |                       | caling                |
|                       |                       |       block: made the |
|                       |                       |       following       |
|                       |                       |       fields          |
|                       |                       |       optional:       |
|                       |                       |       latencyDistribu |
|                       |                       | tion                  |
|                       |                       |       (which is an    |
|                       |                       |       array of        |
|                       |                       |       latencyBucketMe |
|                       |                       | asure                 |
|                       |                       |       structures) and |
|                       |                       |       meanRequestLate |
|                       |                       | ncy.                  |
|                       |                       |       Updated the     |
|                       |                       |       JSON schemas    |
|                       |                       |       (now v24) to    |
|                       |                       |       match.          |
+-----------------------+-----------------------+-----------------------+
| 9/16/2016             | v2.11                 | -  1 Introduction:    |
|                       |                       |       updated the     |
|                       |                       |       introduction to |
|                       |                       |       clarify the     |
|                       |                       |       usage of        |
|                       |                       |       eventTypes and  |
|                       |                       |       the possibility |
|                       |                       |       of support for  |
|                       |                       |       other           |
|                       |                       |       protocols.      |
|                       |                       |                       |
|                       |                       | -  6.1 REST Operation |
|                       |                       |       Overview: added |
|                       |                       |       two new         |
|                       |                       |       subsections     |
|                       |                       |       (6.1.2 and      |
|                       |                       |       6.1.3)          |
|                       |                       |       discussing Api  |
|                       |                       |       Version and     |
|                       |                       |       Commands Toward |
|                       |                       |       Event Source    |
|                       |                       |       Clients.        |
|                       |                       |                       |
|                       |                       | -  6.2                |
|                       |                       |       publishAnyEvent |
|                       |                       | :                     |
|                       |                       |       fixed the       |
|                       |                       |       sample to       |
|                       |                       |       conform to the  |
|                       |                       |       latest changes  |
|                       |                       |                       |
|                       |                       | -  6.3                |
|                       |                       |       publishSpecific |
|                       |                       | Topic:                |
|                       |                       |       fixed the       |
|                       |                       |       sample to       |
|                       |                       |       conform to the  |
|                       |                       |       latest changes  |
|                       |                       |                       |
|                       |                       | -  6.4                |
|                       |                       |       publishEventBat |
|                       |                       | ch:                   |
|                       |                       |       fixed the       |
|                       |                       |       sample to       |
|                       |                       |       conform to the  |
|                       |                       |       latest changes  |
|                       |                       |                       |
|                       |                       | -  6.5                |
|                       |                       |       provideThrottli |
|                       |                       | ngState               |
|                       |                       |       operation:      |
|                       |                       |       added the Input |
|                       |                       |       Parameters      |
|                       |                       |       section heading |
|                       |                       |       back and fixed  |
|                       |                       |       the sample      |
|                       |                       |       request to      |
|                       |                       |       provide         |
|                       |                       |       eventThrottling |
|                       |                       | State                 |
|                       |                       |       (instead of     |
|                       |                       |       eventThrottling |
|                       |                       | ClientState).         |
|                       |                       |                       |
|                       |                       | -  The remaining      |
|                       |                       |       bullets         |
|                       |                       |       describe        |
|                       |                       |       changes made to |
|                       |                       |       section 4       |
|                       |                       |       datatypes in    |
|                       |                       |       alphabetical    |
|                       |                       |       order:          |
|                       |                       |                       |
|                       |                       | -  command datatype:  |
|                       |                       |       referenced the  |
|                       |                       |       new section     |
|                       |                       |       6.1.3 which     |
|                       |                       |       provides an     |
|                       |                       |       explanation of  |
|                       |                       |       command state   |
|                       |                       |       expectations    |
|                       |                       |       and             |
|                       |                       |       requirements    |
|                       |                       |       for a given     |
|                       |                       |       eventSource:    |
|                       |                       |                       |
|                       |                       | -  commonEventHeader  |
|                       |                       |       datatype:       |
|                       |                       |                       |
|                       |                       |    -  made sourceId   |
|                       |                       |          and          |
|                       |                       |          reportingEnt |
|                       |                       | ityId                 |
|                       |                       |          fields       |
|                       |                       |          optional     |
|                       |                       |          (although    |
|                       |                       |          the internal |
|                       |                       |          Generic      |
|                       |                       |          Event        |
|                       |                       |          Listener     |
|                       |                       |          spec         |
|                       |                       |          indicates,   |
|                       |                       |          in the field |
|                       |                       |          descriptions |
|                       |                       | ,                     |
|                       |                       |          that the     |
|                       |                       |          AT&T         |
|                       |                       |          enrichment   |
|                       |                       |          process      |
|                       |                       |          shall ensure |
|                       |                       |          that these   |
|                       |                       |          fields are   |
|                       |                       |          populated)   |
|                       |                       |                       |
|                       |                       |    -  domain          |
|                       |                       |          enumeration: |
|                       |                       |          changed      |
|                       |                       |          measurements |
|                       |                       | ForVfScalingFields    |
|                       |                       |          to           |
|                       |                       |          measurements |
|                       |                       | ForVfScaling          |
|                       |                       |                       |
|                       |                       | -  eventDomainThrottl |
|                       |                       | eSpecificationList:   |
|                       |                       |       added this      |
|                       |                       |       array of        |
|                       |                       |       eventDomainThro |
|                       |                       | ttleSpecification     |
|                       |                       |       stuctures back  |
|                       |                       |       to the schema   |
|                       |                       |       because it is   |
|                       |                       |       used by the     |
|                       |                       |       provideThrottli |
|                       |                       | ngState               |
|                       |                       |       operation.      |
|                       |                       |                       |
|                       |                       | -  eventList: added   |
|                       |                       |       eventList back  |
|                       |                       |       to the vendor   |
|                       |                       |       version of the  |
|                       |                       |       commonEventForm |
|                       |                       | at.                   |
|                       |                       |       This is used by |
|                       |                       |       the             |
|                       |                       |       publishEventBat |
|                       |                       | ch                    |
|                       |                       |       operation.      |
|                       |                       |                       |
|                       |                       | -  faultFields        |
|                       |                       |       datatype:       |
|                       |                       |                       |
|                       |                       |    -  eventSourceType |
|                       |                       | :                     |
|                       |                       |          made this a  |
|                       |                       |          string (and  |
|                       |                       |          provided the |
|                       |                       |          previous     |
|                       |                       |          enumerated   |
|                       |                       |          values as    |
|                       |                       |          examples)    |
|                       |                       |                       |
|                       |                       | -  filesystemUsage    |
|                       |                       |       datatype:       |
|                       |                       |                       |
|                       |                       |    -  changed         |
|                       |                       |          vmIdentifier |
|                       |                       |          to           |
|                       |                       |          filesystemNa |
|                       |                       | me                    |
|                       |                       |                       |
|                       |                       | -  gtpPerFlowMetrics  |
|                       |                       |       datatype:       |
|                       |                       |                       |
|                       |                       |    -  flowActivationT |
|                       |                       | ime:                  |
|                       |                       |          changed the  |
|                       |                       |          format and   |
|                       |                       |          description  |
|                       |                       |          to be        |
|                       |                       |          compliant    |
|                       |                       |          with RFC     |
|                       |                       |          2822.        |
|                       |                       |                       |
|                       |                       |    -  flowDeactivatio |
|                       |                       | nTime:                |
|                       |                       |          changed the  |
|                       |                       |          format and   |
|                       |                       |          description  |
|                       |                       |          to be        |
|                       |                       |          compliant    |
|                       |                       |          with RFC     |
|                       |                       |          2822.        |
|                       |                       |                       |
|                       |                       | -  internalHeaderFiel |
|                       |                       | ds                    |
|                       |                       |       datatype:       |
|                       |                       |                       |
|                       |                       |    -  Added the       |
|                       |                       |          following    |
|                       |                       |          optional     |
|                       |                       |          fields:      |
|                       |                       |          firstDateTim |
|                       |                       | e,                    |
|                       |                       |          lastDateTime |
|                       |                       |          compliant    |
|                       |                       |          with RFC     |
|                       |                       |          2822. Noted  |
|                       |                       |          in the       |
|                       |                       |          description  |
|                       |                       |          that these   |
|                       |                       |          fields must  |
|                       |                       |          be supplied  |
|                       |                       |          for events   |
|                       |                       |          in the       |
|                       |                       |          following    |
|                       |                       |          domains:     |
|                       |                       |          fault,       |
|                       |                       |          thresholdCro |
|                       |                       | ssingAlerts           |
|                       |                       |          and          |
|                       |                       |          measurements |
|                       |                       | ForVfScaling.         |
|                       |                       |                       |
|                       |                       |    -  ticketingTimest |
|                       |                       | amp:                  |
|                       |                       |          changed the  |
|                       |                       |          format and   |
|                       |                       |          description  |
|                       |                       |          to be        |
|                       |                       |          compliant    |
|                       |                       |          with RFC     |
|                       |                       |          2822.        |
|                       |                       |                       |
|                       |                       | -  syslogFields       |
|                       |                       |       datatype:       |
|                       |                       |                       |
|                       |                       |    -  eventSourceType |
|                       |                       | :                     |
|                       |                       |          made this a  |
|                       |                       |          string (and  |
|                       |                       |          provided the |
|                       |                       |          previous     |
|                       |                       |          enumerated   |
|                       |                       |          values,      |
|                       |                       |          without the  |
|                       |                       |          numbers, as  |
|                       |                       |          examples)    |
|                       |                       |                       |
|                       |                       | -  thresholdCrossingA |
|                       |                       | lerts                 |
|                       |                       |       dataypte:       |
|                       |                       |                       |
|                       |                       |    -  collectionTimes |
|                       |                       | tamp:                 |
|                       |                       |          changed the  |
|                       |                       |          format and   |
|                       |                       |          description  |
|                       |                       |          to be        |
|                       |                       |          compliant    |
|                       |                       |          with RFC     |
|                       |                       |          2822.        |
|                       |                       |                       |
|                       |                       |    -  eventStartTimes |
|                       |                       | tamp:                 |
|                       |                       |          changed the  |
|                       |                       |          format and   |
|                       |                       |          description  |
|                       |                       |          to be        |
|                       |                       |          compliant    |
|                       |                       |          with RFC     |
|                       |                       |          2822.        |
|                       |                       |                       |
|                       |                       |    -  added the same  |
|                       |                       |          eventSeverit |
|                       |                       | y                     |
|                       |                       |          field as     |
|                       |                       |          from the     |
|                       |                       |          faultFields  |
|                       |                       |          and made it  |
|                       |                       |          required     |
+-----------------------+-----------------------+-----------------------+
| 9/23/2016             | v2.12                 | -  Section 4          |
|                       |                       |       Datatypes:      |
|                       |                       |       commonEventHead |
|                       |                       | er:                   |
|                       |                       |       made            |
|                       |                       |       reportingEntity |
|                       |                       | Name                  |
|                       |                       |       a required      |
|                       |                       |       field (note:    |
|                       |                       |       the JSON schema |
|                       |                       |       already had     |
|                       |                       |       this field as   |
|                       |                       |       required)       |
+-----------------------+-----------------------+-----------------------+
| 11/29/2016            | v3.0                  | -  Introduction:      |
|                       |                       |                       |
|                       |                       |    -  Introductory    |
|                       |                       |          paragraph:   |
|                       |                       |          changed      |
|                       |                       |          '...Common   |
|                       |                       |          Event Header |
|                       |                       |          Block        |
|                       |                       |          followed by  |
|                       |                       |          zero or more |
|                       |                       |          event domain |
|                       |                       |          blocks' to   |
|                       |                       |          '...Common   |
|                       |                       |          Event Header |
|                       |                       |          Block        |
|                       |                       |          accompanied  |
|                       |                       |          by zero or   |
|                       |                       |          more event   |
|                       |                       |          domain       |
|                       |                       |          blocks'      |
|                       |                       |          since the    |
|                       |                       |          order of the |
|                       |                       |          blocks on    |
|                       |                       |          the wire is  |
|                       |                       |          not          |
|                       |                       |          guaranteed.  |
|                       |                       |                       |
|                       |                       |    -  Added Section   |
|                       |                       |          1.5          |
|                       |                       |          Versioning   |
|                       |                       |                       |
|                       |                       | -  Section 4: codec   |
|                       |                       |       processing:     |
|                       |                       |                       |
|                       |                       |    -  CommonEventForm |
|                       |                       | at_Vendors            |
|                       |                       |          schema only: |
|                       |                       |          codesInUse:  |
|                       |                       |          changed      |
|                       |                       |          required     |
|                       |                       |          field from   |
|                       |                       |          "codecUtiliz |
|                       |                       | ation"                |
|                       |                       |          which was    |
|                       |                       |          removed      |
|                       |                       |          previously   |
|                       |                       |          to           |
|                       |                       |          "numberInUse |
|                       |                       | "                     |
|                       |                       |          which is the |
|                       |                       |          new field    |
|                       |                       |          name.        |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘codecSelect |
|                       |                       | ed’                   |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘codecSelect |
|                       |                       | edTranscoding’        |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       | -  Section 4 and      |
|                       |                       |       section 6:      |
|                       |                       |       command         |
|                       |                       |       processing:     |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          commandListE |
|                       |                       | ntry                  |
|                       |                       |          which is an  |
|                       |                       |          object that  |
|                       |                       |          references   |
|                       |                       |          the command  |
|                       |                       |          object.      |
|                       |                       |                       |
|                       |                       |    -  commandList:    |
|                       |                       |          changed      |
|                       |                       |          commandList  |
|                       |                       |          to contain   |
|                       |                       |          an array of  |
|                       |                       |          commandListE |
|                       |                       | ntry                  |
|                       |                       |          objects.     |
|                       |                       |                       |
|                       |                       |    -  Updated sample  |
|                       |                       |          responses in |
|                       |                       |          section 6    |
|                       |                       |          where        |
|                       |                       |          commands are |
|                       |                       |          used         |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |       commonEventHead |
|                       |                       | er:                   |
|                       |                       |                       |
|                       |                       |    -  Incremented     |
|                       |                       |          version to   |
|                       |                       |          1.2          |
|                       |                       |                       |
|                       |                       |    -  added two new   |
|                       |                       |          values to    |
|                       |                       |          the ‘domain’ |
|                       |                       |          enumeration: |
|                       |                       |          ‘serviceEven |
|                       |                       | ts’                   |
|                       |                       |          and          |
|                       |                       |          ‘signaling   |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |       endOfCallVqmSum |
|                       |                       | maries                |
|                       |                       |       datatype        |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |       ‘event’: added  |
|                       |                       |       two fields:     |
|                       |                       |       ‘serviceEventsF |
|                       |                       | ields’                |
|                       |                       |       and             |
|                       |                       |       ‘signalingField |
|                       |                       | s’                    |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |       ‘eventInstanceI |
|                       |                       | dentifier’datatype    |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |       CommonEventList |
|                       |                       | ener                  |
|                       |                       |       only:           |
|                       |                       |       internalHeaderF |
|                       |                       | ields:                |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘internalHea |
|                       |                       | derFieldsVersion’(ini |
|                       |                       | tially                |
|                       |                       |          set to 1.1)  |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘correlation |
|                       |                       | FirstEpoch’           |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'closedLoopC |
|                       |                       | ontrolName'           |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'closedLoopF |
|                       |                       | lag'                  |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'collectorTi |
|                       |                       | meStamp'              |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'eventTag'   |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘tenantName’ |
|                       |                       |                       |
|                       |                       |    -  changed         |
|                       |                       |          'operational |
|                       |                       | Status'               |
|                       |                       |          to 'inMaint' |
|                       |                       |                       |
|                       |                       |    -  added required  |
|                       |                       |          fields in    |
|                       |                       |          the schema   |
|                       |                       |          to match the |
|                       |                       |          word doc:    |
|                       |                       |          'equipmentNa |
|                       |                       | meCode',              |
|                       |                       |          'equipmentTy |
|                       |                       | pe',                  |
|                       |                       |          'equipmentVe |
|                       |                       | ndor',                |
|                       |                       |          'inMaint',   |
|                       |                       |          'provStatus' |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |       ‘marker’datatyp |
|                       |                       | e                     |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |       ‘midCallRtcp’   |
|                       |                       |       datatype        |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |       mobileFlowField |
|                       |                       | s:                    |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘mobileFlowF |
|                       |                       | ieldsVersion’(initial |
|                       |                       | ly                    |
|                       |                       |          set to 1.1)  |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |       ‘serviceEventsF |
|                       |                       | ields’datatype        |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |       ‘signalingField |
|                       |                       | s’                    |
|                       |                       |       datatype        |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |       syslogFields:   |
|                       |                       |                       |
|                       |                       |    -  Incremented     |
|                       |                       |          syslogFields |
|                       |                       | Version               |
|                       |                       |          to 1.2       |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'syslogPri'  |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'syslogSev'  |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          ‘syslogSdId’ |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |       thresholdCrossi |
|                       |                       | ngAlertFields:        |
|                       |                       |                       |
|                       |                       |    -  Incremented     |
|                       |                       |          thresholdCro |
|                       |                       | ssingFieldsVersion    |
|                       |                       |          to 1.2       |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          'additionalF |
|                       |                       | ields'                |
|                       |                       |          which is an  |
|                       |                       |          optional     |
|                       |                       |          list of name |
|                       |                       |          value pairs. |
|                       |                       |                       |
|                       |                       | -  Section 4: schema  |
|                       |                       |       v26.0 embedded  |
|                       |                       |       reflecting the  |
|                       |                       |       above changes.  |
|                       |                       |                       |
|                       |                       | -  Section 6 and      |
|                       |                       |       Section 2:      |
|                       |                       |       changed all     |
|                       |                       |       sample requests |
|                       |                       |       to use /v3 in   |
|                       |                       |       the REST        |
|                       |                       |       Resource URL.   |
+-----------------------+-----------------------+-----------------------+
| 12/1/2016             | v3.1                  | -  Section 6: Updated |
|                       |                       |       the call flow   |
|                       |                       |       diagrams to     |
|                       |                       |       show ‘v3’       |
+-----------------------+-----------------------+-----------------------+
| 1/5/2017              | v4.0                  | -  Combined the       |
|                       |                       |       Generic Event   |
|                       |                       |       Listener and    |
|                       |                       |       Vendor Event    |
|                       |                       |       Listener into a |
|                       |                       |       single API      |
|                       |                       |       service         |
|                       |                       |       specification   |
|                       |                       |       with version    |
|                       |                       |       4.0.            |
|                       |                       |                       |
|                       |                       | -  Changed the title  |
|                       |                       |       to VES (Virtual |
|                       |                       |       Function Event  |
|                       |                       |       Streaming)      |
|                       |                       |       Listener.       |
|                       |                       |                       |
|                       |                       | -  Changed references |
|                       |                       |       to 'generic     |
|                       |                       |       event' to       |
|                       |                       |       'common event'  |
|                       |                       |       or 'VES event'  |
|                       |                       |       (depending on   |
|                       |                       |       the context)    |
|                       |                       |       throughout the  |
|                       |                       |       document.       |
|                       |                       |                       |
|                       |                       | -  Used the Legal     |
|                       |                       |       Disclaimer from |
|                       |                       |       the Vendor      |
|                       |                       |       Event Listener  |
|                       |                       |       on the back of  |
|                       |                       |       the title page. |
|                       |                       |                       |
|                       |                       | -  Section 1:         |
|                       |                       |       Introduction    |
|                       |                       |       changes:        |
|                       |                       |                       |
|                       |                       |    -  modified        |
|                       |                       |          wording to   |
|                       |                       |          reference    |
|                       |                       |          'VES'        |
|                       |                       |                       |
|                       |                       |    -  removed the     |
|                       |                       |          'Audience'   |
|                       |                       |          section,     |
|                       |                       |          which        |
|                       |                       |          described    |
|                       |                       |          various AT&T |
|                       |                       |          groups the   |
|                       |                       |          documented   |
|                       |                       |          was intended |
|                       |                       |          for          |
|                       |                       |                       |
|                       |                       |    -  tweaked the     |
|                       |                       |          naming       |
|                       |                       |          standards    |
|                       |                       |          for event    |
|                       |                       |          types to     |
|                       |                       |          clarify the  |
|                       |                       |          purpose of   |
|                       |                       |          the naming   |
|                       |                       |          conventions  |
|                       |                       |                       |
|                       |                       | -  Section 3:         |
|                       |                       |       Resource        |
|                       |                       |       Structure:      |
|                       |                       |       added a         |
|                       |                       |       sentence        |
|                       |                       |       describing the  |
|                       |                       |       FQDN and port   |
|                       |                       |       used in the     |
|                       |                       |       resource URL.   |
|                       |                       |                       |
|                       |                       | -  Section 4: Common  |
|                       |                       |       Event Format    |
|                       |                       |       changes:        |
|                       |                       |                       |
|                       |                       |    -  renamed the     |
|                       |                       |          section to   |
|                       |                       |          'Common      |
|                       |                       |          Event        |
|                       |                       |          Format' from |
|                       |                       |          'Generic     |
|                       |                       |          Event        |
|                       |                       |          Format'      |
|                       |                       |                       |
|                       |                       |    -  reorganized the |
|                       |                       |          datatypes    |
|                       |                       |          into         |
|                       |                       |          separate     |
|                       |                       |          sections;    |
|                       |                       |          sections     |
|                       |                       |          were defined |
|                       |                       |          for each of  |
|                       |                       |          the domains  |
|                       |                       |          as well as   |
|                       |                       |          for common   |
|                       |                       |          event,       |
|                       |                       |          common event |
|                       |                       |          header and   |
|                       |                       |          command list |
|                       |                       |          processing   |
|                       |                       |                       |
|                       |                       |    -  codecSelected   |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  codecSelectedTr |
|                       |                       | anscoding             |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  command         |
|                       |                       |          datatype:    |
|                       |                       |          added an     |
|                       |                       |          enumerated   |
|                       |                       |          value to     |
|                       |                       |          commandType: |
|                       |                       |          'heartbeatIn |
|                       |                       | tervalChange'         |
|                       |                       |                       |
|                       |                       |    -  commonEventHead |
|                       |                       | er:                   |
|                       |                       |          added        |
|                       |                       |          internalHead |
|                       |                       | erFields              |
|                       |                       |          to the       |
|                       |                       |          commonEventH |
|                       |                       | eader,                |
|                       |                       |          defined as   |
|                       |                       |          "Fields (not |
|                       |                       |          supplied by  |
|                       |                       |          event        |
|                       |                       |          sources)     |
|                       |                       |          that the VES |
|                       |                       |          Event        |
|                       |                       |          Listener     |
|                       |                       |          service can  |
|                       |                       |          use to       |
|                       |                       |          enrich the   |
|                       |                       |          event if     |
|                       |                       |          needed for   |
|                       |                       |          efficient    |
|                       |                       |          internal     |
|                       |                       |          processing.  |
|                       |                       |          This is an   |
|                       |                       |          empty object |
|                       |                       |          which is     |
|                       |                       |          intended to  |
|                       |                       |          be defined   |
|                       |                       |          separately   |
|                       |                       |          by each      |
|                       |                       |          provider     |
|                       |                       |          implementing |
|                       |                       |          the VES      |
|                       |                       |          Event        |
|                       |                       |          Listener."   |
|                       |                       |                       |
|                       |                       |    -  commonEventHead |
|                       |                       | er:                   |
|                       |                       |          removed two  |
|                       |                       |          enumerated   |
|                       |                       |          values,      |
|                       |                       |          'serviceEven |
|                       |                       | ts'                   |
|                       |                       |          and          |
|                       |                       |          'signaling'  |
|                       |                       |          from the     |
|                       |                       |          domain       |
|                       |                       |          enumeration  |
|                       |                       |                       |
|                       |                       |    -  commonEventHead |
|                       |                       | er                    |
|                       |                       |          version:     |
|                       |                       |          incremented  |
|                       |                       |          the version  |
|                       |                       |          to 2.0       |
|                       |                       |                       |
|                       |                       |    -  endOfCallVqmSum |
|                       |                       | maries                |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  event: changed  |
|                       |                       |          the          |
|                       |                       |          description  |
|                       |                       |          of the event |
|                       |                       |          datatype to: |
|                       |                       |          "fields      |
|                       |                       |          which        |
|                       |                       |          constitute   |
|                       |                       |          the ‘root    |
|                       |                       |          level’ of    |
|                       |                       |          the common   |
|                       |                       |          event        |
|                       |                       |          format"      |
|                       |                       |                       |
|                       |                       |    -  event: removed  |
|                       |                       |          'serviceEven |
|                       |                       | tFields'              |
|                       |                       |          and          |
|                       |                       |          'signalingFi |
|                       |                       | elds'                 |
|                       |                       |          from the     |
|                       |                       |          definition   |
|                       |                       |                       |
|                       |                       |    -  event: fixed a  |
|                       |                       |          misspelling  |
|                       |                       |          of           |
|                       |                       |          ‘thresholdCr |
|                       |                       | ossingAlertFields’,   |
|                       |                       |          which was    |
|                       |                       |          only present |
|                       |                       |          in the Word  |
|                       |                       |          document     |
|                       |                       |                       |
|                       |                       |    -  eventInstanceId |
|                       |                       | entifier              |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  internalHeaderF |
|                       |                       | Ields                 |
|                       |                       |          datatype:    |
|                       |                       |          defined this |
|                       |                       |          as follows:  |
|                       |                       |          "The         |
|                       |                       |          internalHead |
|                       |                       | erFields              |
|                       |                       |          datatype is  |
|                       |                       |          an undefined |
|                       |                       |          object which |
|                       |                       |          can contain  |
|                       |                       |          arbitrarily  |
|                       |                       |          complex JSON |
|                       |                       |          structures.  |
|                       |                       |          It is        |
|                       |                       |          intended to  |
|                       |                       |          be defined   |
|                       |                       |          separately   |
|                       |                       |          by each      |
|                       |                       |          provider     |
|                       |                       |          implementing |
|                       |                       |          the VES      |
|                       |                       |          Event        |
|                       |                       |          Listener.    |
|                       |                       |          The fields   |
|                       |                       |          in           |
|                       |                       |          internalHead |
|                       |                       | erFields              |
|                       |                       |          are not      |
|                       |                       |          provided by  |
|                       |                       |          any event    |
|                       |                       |          source but   |
|                       |                       |          instead are  |
|                       |                       |          added by the |
|                       |                       |          VES Event    |
|                       |                       |          Listener     |
|                       |                       |          service      |
|                       |                       |          itself as    |
|                       |                       |          part of an   |
|                       |                       |          event        |
|                       |                       |          enrichment   |
|                       |                       |          process      |
|                       |                       |          necessary    |
|                       |                       |          for          |
|                       |                       |          efficient    |
|                       |                       |          internal     |
|                       |                       |          processing   |
|                       |                       |          of events    |
|                       |                       |          received by  |
|                       |                       |          the VES      |
|                       |                       |          Event        |
|                       |                       |          Listener"    |
|                       |                       |                       |
|                       |                       |    -  marker          |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  measurementsFor |
|                       |                       | VfScalingFields       |
|                       |                       |          datatype:    |
|                       |                       |          clarified    |
|                       |                       |          that         |
|                       |                       |          memoryConfig |
|                       |                       | ured                  |
|                       |                       |          and          |
|                       |                       |          memoryUsed   |
|                       |                       |          are measured |
|                       |                       |          in MB        |
|                       |                       |                       |
|                       |                       |    -  midCallRtcp     |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  mobileFlowField |
|                       |                       | s                     |
|                       |                       |          datatype:    |
|                       |                       |          added        |
|                       |                       |          ‘additionalF |
|                       |                       | ields’                |
|                       |                       |                       |
|                       |                       |    -  mobileFlowField |
|                       |                       | s                     |
|                       |                       |          datatype:    |
|                       |                       |          incremented  |
|                       |                       |          the version  |
|                       |                       |          number for   |
|                       |                       |          this field   |
|                       |                       |          block to 1.2 |
|                       |                       |                       |
|                       |                       |    -  serviceEventsFi |
|                       |                       | elds                  |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  signalingFields |
|                       |                       |          datatype:    |
|                       |                       |          removed this |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  syslogFields:   |
|                       |                       |          added three  |
|                       |                       |          fields to    |
|                       |                       |          the schema   |
|                       |                       |          that were    |
|                       |                       |          previously   |
|                       |                       |          described in |
|                       |                       |          the document |
|                       |                       |          but not      |
|                       |                       |          incorporated |
|                       |                       |          into the     |
|                       |                       |          schema:      |
|                       |                       |          syslogPri,   |
|                       |                       |          syslogSev,   |
|                       |                       |          syslogSdId   |
|                       |                       |                       |
|                       |                       |    -  syslogFields    |
|                       |                       |          version:     |
|                       |                       |          incremented  |
|                       |                       |          the version  |
|                       |                       |          to 2.0       |
|                       |                       |                       |
|                       |                       | -  Modified the       |
|                       |                       |       Common Event    |
|                       |                       |       Format JSON     |
|                       |                       |       schema to v27.0 |
|                       |                       |       to incorporate  |
|                       |                       |       the above       |
|                       |                       |       changes. Also,  |
|                       |                       |       added the AT&T  |
|                       |                       |       Copyright       |
|                       |                       |       Notice from the |
|                       |                       |       top of the      |
|                       |                       |       retired         |
|                       |                       |       CommonEventForm |
|                       |                       | at_Vendors            |
|                       |                       |       schema.         |
|                       |                       |                       |
|                       |                       | -  Section 6 and 2:   |
|                       |                       |       changed all     |
|                       |                       |       sample requests |
|                       |                       |       to use /v4 in   |
|                       |                       |       the REST        |
|                       |                       |       Resource URL    |
|                       |                       |       and call flow   |
|                       |                       |       diagrams.       |
|                       |                       |                       |
|                       |                       | -  Section 6.1.3:     |
|                       |                       |    added a row to the |
|                       |                       |    table in this      |
|                       |                       |    section describing |
|                       |                       |    the                |
|                       |                       |    ‘heartbeatInterval |
|                       |                       | Change’               |
|                       |                       |    command.           |
|                       |                       |                       |
|                       |                       | -  Section 6.1.4:     |
|                       |                       |    added this new     |
|                       |                       |    section describing |
|                       |                       |    expectations for   |
|                       |                       |    buffering of       |
|                       |                       |    events should all  |
|                       |                       |    REST resource URL  |
|                       |                       |    FQDNs be           |
|                       |                       |    unreachable.       |
|                       |                       |                       |
|                       |                       | -  Section 6 Sample   |
|                       |                       |    Requests: modified |
|                       |                       |    all sample         |
|                       |                       |    requests showing   |
|                       |                       |    the return of a    |
|                       |                       |    commandList toward |
|                       |                       |    the event source   |
|                       |                       |    to incorporate a   |
|                       |                       |    heartbeatIntervalC |
|                       |                       | hange                 |
|                       |                       |    command; also      |
|                       |                       |    corrected the      |
|                       |                       |    spelling in the    |
|                       |                       |    samples for the    |
|                       |                       |    measurementInterva |
|                       |                       | lChange               |
|                       |                       |    command.           |
|                       |                       |                       |
|                       |                       | -  Section 7:         |
|                       |                       |       Contributors:   |
|                       |                       |       removed this    |
|                       |                       |       section         |
+-----------------------+-----------------------+-----------------------+
| 3/21/2017             | v4.1                  | -  JSON Schema        |
|                       |                       |    changes to produce |
|                       |                       |    v27.2 (note: an    |
|                       |                       |    earlier draft      |
|                       |                       |    version of v27.1   |
|                       |                       |    had been           |
|                       |                       |    distributed to a   |
|                       |                       |    few individuals):  |
|                       |                       |                       |
|                       |                       |    -  To support use  |
|                       |                       |       of the schema   |
|                       |                       |       with event      |
|                       |                       |       batches,        |
|                       |                       |       removed the     |
|                       |                       |       following       |
|                       |                       |       statement near  |
|                       |                       |       the end of the  |
|                       |                       |       schema file:    |
|                       |                       |                       |
|                       |                       | ..                    |
|                       |                       |                       |
|                       |                       |    “required”: [      |
|                       |                       |    “event” ]          |
|                       |                       |                       |
|                       |                       | -  Fixed the          |
|                       |                       |    characters used in |
|                       |                       |    some of the quotes |
|                       |                       |                       |
|                       |                       | -  Fixed some typos   |
|                       |                       |    in the             |
|                       |                       |    descriptions.      |
|                       |                       |                       |
|                       |                       | -  Removed the        |
|                       |                       |    booleans, which    |
|                       |                       |    were non-essential |
|                       |                       |    and which were     |
|                       |                       |    causing problems   |
|                       |                       |    across different   |
|                       |                       |    implementations.   |
|                       |                       |                       |
|                       |                       | -  Section 4.5.7      |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields:      |
|                       |                       |                       |
|                       |                       |    -  Fixed the       |
|                       |                       |          spelling of  |
|                       |                       |          measurements |
|                       |                       | ForVfScalingFields    |
|                       |                       |          in the Word  |
|                       |                       |          document     |
|                       |                       |                       |
|                       |                       | -  Section 2 and 6    |
|                       |                       |       sample requests |
|                       |                       |       and responses:  |
|                       |                       |                       |
|                       |                       |    -  Removed quotes  |
|                       |                       |          from         |
|                       |                       |          numbers:     |
|                       |                       |          sequence,    |
|                       |                       |          and          |
|                       |                       |          first/lastEp |
|                       |                       | ochMicrosec.          |
|                       |                       |                       |
|                       |                       |    -  Fixed all quote |
|                       |                       |          characters,  |
|                       |                       |          some of      |
|                       |                       |          which were   |
|                       |                       |          using        |
|                       |                       |          unusual      |
|                       |                       |          symbols that |
|                       |                       |          wouldn’t     |
|                       |                       |          validate     |
|                       |                       |          with the     |
|                       |                       |          json-schema  |
|                       |                       |          Python       |
|                       |                       |          package.     |
|                       |                       |                       |
|                       |                       | -  Section 6.2.6.1,   |
|                       |                       |       6.3.6.1,        |
|                       |                       |       6.4.6.1 sample  |
|                       |                       |       requests:       |
|                       |                       |                       |
|                       |                       |    -  Added an        |
|                       |                       |          alarmAdditio |
|                       |                       | nalInformation        |
|                       |                       |          field array  |
|                       |                       |          to the       |
|                       |                       |          sample       |
|                       |                       |          requests.    |
|                       |                       |                       |
|                       |                       |    -  Added missing   |
|                       |                       |          commas.      |
|                       |                       |                       |
|                       |                       | -  Section 6.5.6.1    |
|                       |                       |       provideThrottli |
|                       |                       | ngState               |
|                       |                       |       sample          |
|                       |                       |       requests:       |
|                       |                       |                       |
|                       |                       |    -  Fixed the       |
|                       |                       |          eventDomainT |
|                       |                       | hrottleSpecificationL |
|                       |                       | ist                   |
|                       |                       |          to pass an   |
|                       |                       |          array of     |
|                       |                       |          anonymous    |
|                       |                       |          eventDomainT |
|                       |                       | hrottleSpecification  |
|                       |                       |          objects.     |
|                       |                       |                       |
|                       |                       |    -  Added missing   |
|                       |                       |          quotes.      |
|                       |                       |                       |
|                       |                       | -  Fixed the          |
|                       |                       |       suppressedNvPai |
|                       |                       | rsList                |
|                       |                       |       to pass an      |
|                       |                       |       array of        |
|                       |                       |       anonymous       |
|                       |                       |       suppressedNvPai |
|                       |                       | rs                    |
|                       |                       |       objects.        |
+-----------------------+-----------------------+-----------------------+
| 4/14/2017             | v5.0                  | -  Section 1          |
|                       |                       |    Introduction:      |
|                       |                       |                       |
|                       |                       |    -  Clarified the   |
|                       |                       |       Introduction    |
|                       |                       |       (Section 1).    |
|                       |                       |                       |
|                       |                       |    -  Changed Section |
|                       |                       |       1.1 title from  |
|                       |                       |       ‘Terminology’   |
|                       |                       |       to 'Event       |
|                       |                       |       Registration'   |
|                       |                       |       and referenced  |
|                       |                       |       the YAML event  |
|                       |                       |       registration    |
|                       |                       |       format, defined |
|                       |                       |       in a separate   |
|                       |                       |       document.       |
|                       |                       |                       |
|                       |                       |    -  Clarified       |
|                       |                       |       naming          |
|                       |                       |       standards for   |
|                       |                       |       eventName.      |
|                       |                       |                       |
|                       |                       | -  Section 3: updated |
|                       |                       |       the REST        |
|                       |                       |       resource        |
|                       |                       |       structure       |
|                       |                       |                       |
|                       |                       | -  Section 4.1        |
|                       |                       |       command list    |
|                       |                       |       processing      |
|                       |                       |       datatypes:      |
|                       |                       |                       |
|                       |                       |    -  Got rid of      |
|                       |                       |          commandListE |
|                       |                       | ntry                  |
|                       |                       |          and returned |
|                       |                       |          commandList  |
|                       |                       |          to a simple  |
|                       |                       |          array of     |
|                       |                       |          commands.    |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          heartbeatInt |
|                       |                       | erval                 |
|                       |                       |          to the       |
|                       |                       |          command      |
|                       |                       |          datatype.    |
|                       |                       |                       |
|                       |                       |    -  Changed the     |
|                       |                       |          datatype of  |
|                       |                       |          measurementI |
|                       |                       | nterval               |
|                       |                       |          from number  |
|                       |                       |          to integer.  |
|                       |                       |                       |
|                       |                       | -  Section 4.2 common |
|                       |                       |       event           |
|                       |                       |       datatypes:      |
|                       |                       |                       |
|                       |                       |    -  event dataType: |
|                       |                       |          Added        |
|                       |                       |          heartbeatFie |
|                       |                       | lds,                  |
|                       |                       |          sipSignaling |
|                       |                       | Fields                |
|                       |                       |          and          |
|                       |                       |          voiceQuality |
|                       |                       | Fields                |
|                       |                       |          to the event |
|                       |                       |          datatype as  |
|                       |                       |          optional     |
|                       |                       |          field blocks |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          jsonObject   |
|                       |                       |          which        |
|                       |                       |          provides a   |
|                       |                       |          json object  |
|                       |                       |          schema, name |
|                       |                       |          and other    |
|                       |                       |          meta-informa |
|                       |                       | tion                  |
|                       |                       |          along with   |
|                       |                       |          one or more  |
|                       |                       |          object       |
|                       |                       |          instances.   |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          jsonObjectIn |
|                       |                       | stance                |
|                       |                       |          which        |
|                       |                       |          provides     |
|                       |                       |          meta-informa |
|                       |                       | tion                  |
|                       |                       |          about an     |
|                       |                       |          instance of  |
|                       |                       |          a jsonObject |
|                       |                       |          along with   |
|                       |                       |          the actual   |
|                       |                       |          object       |
|                       |                       |          instance     |
|                       |                       |                       |
|                       |                       |    -  Added the ‘key’ |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  Added the       |
|                       |                       |          namedArrayOf |
|                       |                       | Fields                |
|                       |                       |          datatype     |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          vendorVnfNam |
|                       |                       | eFields               |
|                       |                       |                       |
|                       |                       | -  Section 4.3 common |
|                       |                       |       event header    |
|                       |                       |       fields:         |
|                       |                       |                       |
|                       |                       |    -  Add two new     |
|                       |                       |          enumerations |
|                       |                       |          to domain:   |
|                       |                       |          ‘sipSignalin |
|                       |                       | g’                    |
|                       |                       |          and          |
|                       |                       |          ‘voiceQualit |
|                       |                       | y’                    |
|                       |                       |                       |
|                       |                       |    -  Renamed         |
|                       |                       |          eventType to |
|                       |                       |          eventName.   |
|                       |                       |          Note that    |
|                       |                       |          the original |
|                       |                       |          usage of     |
|                       |                       |          eventType    |
|                       |                       |          was formally |
|                       |                       |          described in |
|                       |                       |          the          |
|                       |                       |          Introduction |
|                       |                       |          back on      |
|                       |                       |          2/11/2016    |
|                       |                       |          with v1.9.   |
|                       |                       |                       |
|                       |                       |    -  Made eventName  |
|                       |                       |          a required   |
|                       |                       |          field        |
|                       |                       |                       |
|                       |                       |    -  Created a new   |
|                       |                       |          field called |
|                       |                       |          eventType    |
|                       |                       |          with a       |
|                       |                       |          meaning that |
|                       |                       |          is different |
|                       |                       |          than the old |
|                       |                       |          eventType.   |
|                       |                       |                       |
|                       |                       |    -  Removed         |
|                       |                       |          functionalRo |
|                       |                       | le,                   |
|                       |                       |          which was    |
|                       |                       |          replaced by  |
|                       |                       |          the          |
|                       |                       |          following    |
|                       |                       |          two fields.  |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          nfNamingCode |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          nfcNamingCod |
|                       |                       | e                     |
|                       |                       |                       |
|                       |                       |    -  Changed version |
|                       |                       |          to 3.0       |
|                       |                       |          (major       |
|                       |                       |          version      |
|                       |                       |          change) and  |
|                       |                       |          made it a    |
|                       |                       |          required     |
|                       |                       |          field        |
|                       |                       |                       |
|                       |                       | -  Section 4.4:       |
|                       |                       |       faultFields:    |
|                       |                       |                       |
|                       |                       |    -  added one       |
|                       |                       |          optional     |
|                       |                       |          field:       |
|                       |                       |          eventCategor |
|                       |                       | y                     |
|                       |                       |                       |
|                       |                       |    -  made            |
|                       |                       |          faultFieldsV |
|                       |                       | ersion                |
|                       |                       |          a required   |
|                       |                       |          field        |
|                       |                       |                       |
|                       |                       |    -  changed         |
|                       |                       |          faultFieldsV |
|                       |                       | ersion                |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          version      |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |    -  fixed a typo on |
|                       |                       |          the spelling |
|                       |                       |          of           |
|                       |                       |          alarmInterfa |
|                       |                       | ceA                   |
|                       |                       |                       |
|                       |                       |    -  clarified field |
|                       |                       |          descriptions |
|                       |                       |                       |
|                       |                       | -  Section 4.5: added |
|                       |                       |       heartbeatFields |
|                       |                       |       datatype which  |
|                       |                       |       can be used to  |
|                       |                       |       communicate     |
|                       |                       |       heartbeatInterv |
|                       |                       | al.                   |
|                       |                       |       Note: this      |
|                       |                       |       change was      |
|                       |                       |       previously made |
|                       |                       |       in v4.2         |
|                       |                       |                       |
|                       |                       | -  Section 4.6        |
|                       |                       |       measurements    |
|                       |                       |       for vf scaling  |
|                       |                       |       datatypes:      |
|                       |                       |       changed the     |
|                       |                       |       following       |
|                       |                       |       datatypes from  |
|                       |                       |       number to       |
|                       |                       |       integer:        |
|                       |                       |                       |
|                       |                       |    -  In              |
|                       |                       |          measurements |
|                       |                       | ForVfScalingFields:   |
|                       |                       |          concurrentSe |
|                       |                       | ssions,               |
|                       |                       |          configuredEn |
|                       |                       | tities,               |
|                       |                       |          numberOfMedi |
|                       |                       | aPortsInUse,          |
|                       |                       |          vnfcScalingM |
|                       |                       | etric                 |
|                       |                       |                       |
|                       |                       |    -  In codecsInUse: |
|                       |                       |          numberInUse  |
|                       |                       |                       |
|                       |                       |    -  In              |
|                       |                       |          featuresInUs |
|                       |                       | e:                    |
|                       |                       |          featureUtili |
|                       |                       | zation                |
|                       |                       |                       |
|                       |                       | -  Section 4.6.2      |
|                       |                       |       modified        |
|                       |                       |       cpuUsage        |
|                       |                       |                       |
|                       |                       | -  Section 4.6.3      |
|                       |                       |       added diskUsage |
|                       |                       |                       |
|                       |                       | -  Section 4.6.7      |
|                       |                       |       measurementsFor |
|                       |                       | VfScalingFields:      |
|                       |                       |                       |
|                       |                       |    -  fixed the       |
|                       |                       |          spelling of  |
|                       |                       |          the          |
|                       |                       |          measurements |
|                       |                       | ForVfScalingFields    |
|                       |                       |          in the Word  |
|                       |                       |          document     |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          additionalFi |
|                       |                       | elds,                 |
|                       |                       |          which is an  |
|                       |                       |          array of     |
|                       |                       |          fields       |
|                       |                       |          (i.e.,       |
|                       |                       |          name-value   |
|                       |                       |          pairs)       |
|                       |                       |                       |
|                       |                       |    -  changed         |
|                       |                       |          additionalMe |
|                       |                       | asurements            |
|                       |                       |          to reference |
|                       |                       |          the common   |
|                       |                       |          datatype     |
|                       |                       |          namedArrayOf |
|                       |                       | Fields                |
|                       |                       |          (instead of  |
|                       |                       |          referencing  |
|                       |                       |          measurementG |
|                       |                       | roup)                 |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          additionalOb |
|                       |                       | jects                 |
|                       |                       |          which is an  |
|                       |                       |          array of     |
|                       |                       |          jsonObjects  |
|                       |                       |          described by |
|                       |                       |          name, keys   |
|                       |                       |          and schema   |
|                       |                       |                       |
|                       |                       |    -  deleted         |
|                       |                       |          aggregateCpu |
|                       |                       | Usage                 |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          diskUsageArr |
|                       |                       | ay                    |
|                       |                       |                       |
|                       |                       |    -  deleted         |
|                       |                       |          measurementG |
|                       |                       | roup                  |
|                       |                       |          (which was   |
|                       |                       |          replaced by  |
|                       |                       |          the common   |
|                       |                       |          datatype:    |
|                       |                       |          namedArrayOf |
|                       |                       | Fields                |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          memoryUsageA |
|                       |                       | rray                  |
|                       |                       |                       |
|                       |                       |    -  deleted         |
|                       |                       |          memoryConfig |
|                       |                       | ured                  |
|                       |                       |          and          |
|                       |                       |          memoryUsed   |
|                       |                       |                       |
|                       |                       |    -  deleted errors  |
|                       |                       |          and          |
|                       |                       |          vNicUsageArr |
|                       |                       | ay                    |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |          vNicPerforma |
|                       |                       | nceArray              |
|                       |                       |                       |
|                       |                       |    -  changed the     |
|                       |                       |          measurements |
|                       |                       | ForVfScalingVersion   |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          version      |
|                       |                       |          change) and  |
|                       |                       |          made it a    |
|                       |                       |          required     |
|                       |                       |          field. Also  |
|                       |                       |          changed the  |
|                       |                       |          name of this |
|                       |                       |          version      |
|                       |                       |          field in the |
|                       |                       |          Word         |
|                       |                       |          document to  |
|                       |                       |          match that   |
|                       |                       |          in the JSON  |
|                       |                       |          schema.      |
|                       |                       |                       |
|                       |                       | -  Section 4.6.8      |
|                       |                       |       added           |
|                       |                       |       memoryUsage     |
|                       |                       |                       |
|                       |                       | -  Section 4.6.9      |
|                       |                       |       vNicPerformance |
|                       |                       | :                     |
|                       |                       |       replaced        |
|                       |                       |       vNicUsage and   |
|                       |                       |       errors with     |
|                       |                       |       vNicPerformance |
|                       |                       |                       |
|                       |                       | -  Section 4.7 mobile |
|                       |                       |       flow fields     |
|                       |                       |       changes:        |
|                       |                       |                       |
|                       |                       |    -  Made            |
|                       |                       |          mobileFlowFi |
|                       |                       | eldsVersion           |
|                       |                       |          a required   |
|                       |                       |          field and    |
|                       |                       |          changed the  |
|                       |                       |          mobileFlowFi |
|                       |                       | eldsVersion           |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          version      |
|                       |                       |          change).     |
|                       |                       |                       |
|                       |                       |    -  Changed the     |
|                       |                       |          datatype of  |
|                       |                       |          flowActivati |
|                       |                       | onTime                |
|                       |                       |          and          |
|                       |                       |          flowDeactiva |
|                       |                       | tionTime              |
|                       |                       |          in the Word  |
|                       |                       |          doc to       |
|                       |                       |          string.      |
|                       |                       |                       |
|                       |                       |    -  changed the     |
|                       |                       |          following    |
|                       |                       |          datatypes    |
|                       |                       |          from number  |
|                       |                       |          to integer:  |
|                       |                       |          otherEndpoin |
|                       |                       | tPort,                |
|                       |                       |          reportingEnd |
|                       |                       | pointPort,            |
|                       |                       |          samplingAlgo |
|                       |                       | rithm                 |
|                       |                       |                       |
|                       |                       | -  Section 4.8:       |
|                       |                       |       otherFields:    |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          otherFieldsV |
|                       |                       | ersion                |
|                       |                       |          (set at 1.1) |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          hashOfNameVa |
|                       |                       | luePairArrays         |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          jsonObjects  |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          nameValuePai |
|                       |                       | rs                    |
|                       |                       |                       |
|                       |                       | -  Section 4.9: added |
|                       |                       |       sipSignaling    |
|                       |                       |       domain          |
|                       |                       |       datatypes with  |
|                       |                       |       4.8.1           |
|                       |                       |       sipSignalingFie |
|                       |                       | lds.                  |
|                       |                       |       sipSignalingFie |
|                       |                       | ldsVersion            |
|                       |                       |       is set at 1.0   |
|                       |                       |                       |
|                       |                       | -  Section 4.10       |
|                       |                       |       stateChangeFiel |
|                       |                       | ds:                   |
|                       |                       |       made            |
|                       |                       |       stateChangeFiel |
|                       |                       | dsVersion             |
|                       |                       |       a required      |
|                       |                       |       field and set   |
|                       |                       |       it to 2.0       |
|                       |                       |       (major version  |
|                       |                       |       change).        |
|                       |                       |                       |
|                       |                       | -  Section 4.11       |
|                       |                       |       syslogFields:   |
|                       |                       |                       |
|                       |                       |    -  Changed the     |
|                       |                       |          following    |
|                       |                       |          datatypes    |
|                       |                       |          from number  |
|                       |                       |          to integer:  |
|                       |                       |          syslogFacili |
|                       |                       | ty,                   |
|                       |                       |          syslogPri    |
|                       |                       |                       |
|                       |                       |    -  Changed         |
|                       |                       |          additionalFi |
|                       |                       | elds                  |
|                       |                       |          from a field |
|                       |                       |          [ ] to a     |
|                       |                       |          string which |
|                       |                       |          takes        |
|                       |                       |          name=value   |
|                       |                       |          pairs        |
|                       |                       |          delimited by |
|                       |                       |          a pipe       |
|                       |                       |          symbol.      |
|                       |                       |                       |
|                       |                       |    -  Changed         |
|                       |                       |          syslogFields |
|                       |                       | Version               |
|                       |                       |          to 3.0       |
|                       |                       |          (major       |
|                       |                       |          version      |
|                       |                       |          change) and  |
|                       |                       |          made it a    |
|                       |                       |          required     |
|                       |                       |          field        |
|                       |                       |                       |
|                       |                       |    -  Made syslogSev  |
|                       |                       |          an           |
|                       |                       |          enumerated   |
|                       |                       |          string       |
|                       |                       |          (previously  |
|                       |                       |          just a       |
|                       |                       |          string)      |
|                       |                       |                       |
|                       |                       | -  Section 4.12       |
|                       |                       |       thresholdCrossi |
|                       |                       | ngAlertFields:        |
|                       |                       |       made            |
|                       |                       |       thresholdCrossi |
|                       |                       | ngFieldsVersion       |
|                       |                       |       a required      |
|                       |                       |       field and set   |
|                       |                       |       it to 2.0       |
|                       |                       |       (major version  |
|                       |                       |       change).        |
|                       |                       |                       |
|                       |                       | -  Section 4.132:     |
|                       |                       |       added voice     |
|                       |                       |       quality domain  |
|                       |                       |       datatypes with  |
|                       |                       |       4.13.1          |
|                       |                       |       endOfCallVqmSum |
|                       |                       | maries                |
|                       |                       |       and 4.13.2      |
|                       |                       |       voiceQualityFie |
|                       |                       | lds.                  |
|                       |                       |       voiceQualityFie |
|                       |                       | ldsVersion            |
|                       |                       |       is set at 1.0   |
|                       |                       |                       |
|                       |                       | -  JSON Schema:       |
|                       |                       |    changed the schema |
|                       |                       |    to v28.0 and       |
|                       |                       |    incorporated all   |
|                       |                       |    of the changes     |
|                       |                       |    above.             |
|                       |                       |                       |
|                       |                       | -  Additional JSON    |
|                       |                       |    Schema changes     |
|                       |                       |    that are part of   |
|                       |                       |    v28: Note: The     |
|                       |                       |    following changes  |
|                       |                       |    are provided       |
|                       |                       |    relative to API    |
|                       |                       |    Spec v4.0 (which   |
|                       |                       |    embedded JSON      |
|                       |                       |    schema v27.0), but |
|                       |                       |    they were also     |
|                       |                       |    made in an interim |
|                       |                       |    release v4.1       |
|                       |                       |    (which embedded    |
|                       |                       |    JSON schema        |
|                       |                       |    v27.2):            |
|                       |                       |                       |
|                       |                       |    -  To support use  |
|                       |                       |       of the schema   |
|                       |                       |       with event      |
|                       |                       |       batches,        |
|                       |                       |       removed the     |
|                       |                       |       following       |
|                       |                       |       statement near  |
|                       |                       |       the end of the  |
|                       |                       |       schema file:    |
|                       |                       |                       |
|                       |                       | ..                    |
|                       |                       |                       |
|                       |                       |    “required”: [      |
|                       |                       |    “event” ]          |
|                       |                       |                       |
|                       |                       | -  Fixed the          |
|                       |                       |    characters used in |
|                       |                       |    some of the quotes |
|                       |                       |                       |
|                       |                       | -  Fixed some typos   |
|                       |                       |    in the             |
|                       |                       |    descriptions.      |
|                       |                       |                       |
|                       |                       | -  Removed the        |
|                       |                       |    booleans, which    |
|                       |                       |    were non-essential |
|                       |                       |    and which were     |
|                       |                       |    causing problems   |
|                       |                       |    across different   |
|                       |                       |    implementations.   |
|                       |                       |                       |
|                       |                       | -  Section 2 and 6    |
|                       |                       |       sample requests |
|                       |                       |       and responses   |
|                       |                       |       (also           |
|                       |                       |       incorporated in |
|                       |                       |       interim release |
|                       |                       |       4.1):           |
|                       |                       |                       |
|                       |                       |    -  Removed quotes  |
|                       |                       |          from         |
|                       |                       |          numbers:     |
|                       |                       |          sequence,    |
|                       |                       |          and          |
|                       |                       |          first/lastEp |
|                       |                       | ochMicrosec.          |
|                       |                       |                       |
|                       |                       |    -  Fixed all quote |
|                       |                       |          characters,  |
|                       |                       |          some of      |
|                       |                       |          which were   |
|                       |                       |          using        |
|                       |                       |          unusual      |
|                       |                       |          symbols that |
|                       |                       |          wouldn’t     |
|                       |                       |          validate     |
|                       |                       |          with the     |
|                       |                       |          json-schema  |
|                       |                       |          Python       |
|                       |                       |          package.     |
|                       |                       |                       |
|                       |                       | -  Section 2 and 6    |
|                       |                       |       sample requests |
|                       |                       |       and responses   |
|                       |                       |       (only in v5.0): |
|                       |                       |                       |
|                       |                       |    -  Changed the     |
|                       |                       |          version      |
|                       |                       |          numbers in   |
|                       |                       |          the URL      |
|                       |                       |          string.      |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |          nfNamingCode |
|                       |                       |          and          |
|                       |                       |          nfcNamingCod |
|                       |                       | e                     |
|                       |                       |          and removed  |
|                       |                       |          functionalRo |
|                       |                       | le                    |
|                       |                       |                       |
|                       |                       | -  Section 6 call     |
|                       |                       |       flows: updated  |
|                       |                       |       the version     |
|                       |                       |       number (only in |
|                       |                       |       v5.0).          |
|                       |                       |                       |
|                       |                       | -  Section 6: removed |
|                       |                       |       the             |
|                       |                       |       publishSpecific |
|                       |                       | Topic                 |
|                       |                       |       operation       |
|                       |                       |                       |
|                       |                       | -  Section 6.1.4:     |
|                       |                       |       Buffering:      |
|                       |                       |       clarified event |
|                       |                       |       source          |
|                       |                       |       expectations    |
|                       |                       |       for buffering   |
|                       |                       |       (only in v5.0). |
|                       |                       |                       |
|                       |                       | -  Section 6.2.6.1,   |
|                       |                       |       6.3.6.1 sample  |
|                       |                       |       requests (also  |
|                       |                       |       incorporated in |
|                       |                       |       interim release |
|                       |                       |       4.1):           |
|                       |                       |                       |
|                       |                       |    -  Added an        |
|                       |                       |          alarmAdditio |
|                       |                       | nalInformation        |
|                       |                       |          field array  |
|                       |                       |          to the       |
|                       |                       |          sample       |
|                       |                       |          requests.    |
|                       |                       |                       |
|                       |                       |    -  Added missing   |
|                       |                       |          commas.      |
|                       |                       |                       |
|                       |                       | -  Section 6.2.6.3,   |
|                       |                       |       6.3.6.3         |
|                       |                       |       commandList     |
|                       |                       |       sample          |
|                       |                       |       responses (only |
|                       |                       |       in v5.0):       |
|                       |                       |                       |
|                       |                       |    -  Fixed the       |
|                       |                       |          commandList  |
|                       |                       |          sample       |
|                       |                       |          responses to |
|                       |                       |          pass an      |
|                       |                       |          array of     |
|                       |                       |          anonymous    |
|                       |                       |          command      |
|                       |                       |          objects      |
|                       |                       |          (rather than |
|                       |                       |          an array of  |
|                       |                       |          commandListE |
|                       |                       | ntry                  |
|                       |                       |          objects).    |
|                       |                       |                       |
|                       |                       |    -  Fixed the       |
|                       |                       |          heartbeatInt |
|                       |                       | ervalChange           |
|                       |                       |          commandType  |
|                       |                       |          to pass a    |
|                       |                       |          heartbeatInt |
|                       |                       | erval                 |
|                       |                       |          value        |
|                       |                       |          instead of a |
|                       |                       |          measurementI |
|                       |                       | nterval               |
|                       |                       |          value.       |
|                       |                       |                       |
|                       |                       |    -  Removed quotes  |
|                       |                       |          from the     |
|                       |                       |          measurementI |
|                       |                       | nterval               |
|                       |                       |          and          |
|                       |                       |          heartbeatInt |
|                       |                       | erval                 |
|                       |                       |          values since |
|                       |                       |          they are     |
|                       |                       |          numbers.     |
|                       |                       |                       |
|                       |                       | -  Section 6.4.6.1    |
|                       |                       |       provideThrottli |
|                       |                       | ngState               |
|                       |                       |       sample requests |
|                       |                       |       (also           |
|                       |                       |       incorporated in |
|                       |                       |       interim release |
|                       |                       |       4.1):           |
|                       |                       |                       |
|                       |                       |    -  Fixed the       |
|                       |                       |          eventDomainT |
|                       |                       | hrottleSpecificationL |
|                       |                       | ist                   |
|                       |                       |          to pass an   |
|                       |                       |          array of     |
|                       |                       |          anonymous    |
|                       |                       |          eventDomainT |
|                       |                       | hrottleSpecification  |
|                       |                       |          objects.     |
|                       |                       |                       |
|                       |                       |    -  Added missing   |
|                       |                       |          quotes.      |
|                       |                       |                       |
|                       |                       |    -  Fixed the       |
|                       |                       |       suppressedNvPai |
|                       |                       | rsList                |
|                       |                       |       to pass an      |
|                       |                       |       array of        |
|                       |                       |       anonymous       |
|                       |                       |       suppressedNvPai |
|                       |                       | rs                    |
|                       |                       |       objects (also   |
|                       |                       |       incorporated in |
|                       |                       |       interim release |
|                       |                       |       4.1).           |
+-----------------------+-----------------------+-----------------------+
| 5/22/2017             | v5.1                  | -  Footers: removed   |
|                       |                       |    proprietary        |
|                       |                       |    markings and       |
|                       |                       |    updated copyrights |
|                       |                       |    to 2017            |
|                       |                       |                       |
|                       |                       | -  Section 4.2.3:     |
|                       |                       |    field:             |
|                       |                       |                       |
|                       |                       |    -  Changed the API |
|                       |                       |       spec to make    |
|                       |                       |       ‘name’ and      |
|                       |                       |       ‘value’ start   |
|                       |                       |       with lowercase  |
|                       |                       |       letters. Note:  |
|                       |                       |       this did not    |
|                       |                       |       affect the      |
|                       |                       |       schema, which   |
|                       |                       |       already had     |
|                       |                       |       them as         |
|                       |                       |       lowercase.      |
|                       |                       |                       |
|                       |                       | -  JSON Schema:       |
|                       |                       |                       |
|                       |                       |    -  measurementGrou |
|                       |                       | p:                    |
|                       |                       |       deleted this    |
|                       |                       |       object since it |
|                       |                       |       was replaced    |
|                       |                       |       with            |
|                       |                       |       ‘namedArrayOfFi |
|                       |                       | elds’                 |
|                       |                       |       in v28.0 and    |
|                       |                       |       was no longer   |
|                       |                       |       being used.     |
|                       |                       |                       |
|                       |                       |    -  namedArrayOfFie |
|                       |                       | lds:                  |
|                       |                       |       Fixed an error  |
|                       |                       |       in the          |
|                       |                       |       specification   |
|                       |                       |       of required     |
|                       |                       |       fields: from    |
|                       |                       |       ‘measurements’  |
|                       |                       |       to              |
|                       |                       |       ‘arrayOfFields’ |
|                       |                       | .                     |
|                       |                       |                       |
|                       |                       | -  Changed the        |
|                       |                       |    version of the     |
|                       |                       |    JSON schema to     |
|                       |                       |    28.1               |
+-----------------------+-----------------------+-----------------------+
| 6/14/2017             | v5.2                  | -  JSON Schema:       |
|                       |                       |    created v28.2 by   |
|                       |                       |    changing the field |
|                       |                       |    descriptions in    |
|                       |                       |    the memoryUsage    |
|                       |                       |    object to refer to |
|                       |                       |    ‘kibibytes’        |
|                       |                       |    instead of         |
|                       |                       |    ‘kilobytes’. There |
|                       |                       |    were no changes to |
|                       |                       |    the 28.1           |
|                       |                       |    structure.         |
|                       |                       |                       |
|                       |                       | -  Word Document:     |
|                       |                       |    measurementsForVfS |
|                       |                       | caling                |
|                       |                       |    Domain:            |
|                       |                       |    memoryUsage        |
|                       |                       |    object: changed    |
|                       |                       |    the field          |
|                       |                       |    descriptions in    |
|                       |                       |    this object to     |
|                       |                       |    refer to           |
|                       |                       |    ‘kibibytes’        |
|                       |                       |    instead of         |
|                       |                       |    ‘kilobytes’. There |
|                       |                       |    were no changes to |
|                       |                       |    the memoryUsage    |
|                       |                       |    structure.         |
|                       |                       |                       |
|                       |                       | -  Reorganized the    |
|                       |                       |    Word document to   |
|                       |                       |    group the data     |
|                       |                       |    structures in      |
|                       |                       |    Section 4 into     |
|                       |                       |    three broad        |
|                       |                       |    categories to      |
|                       |                       |    better align with  |
|                       |                       |    the VNF Guidelines |
|                       |                       |    documentation that |
|                       |                       |    has been prepared  |
|                       |                       |    for vendors:       |
|                       |                       |                       |
|                       |                       |    -  Common Event    |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  Command List |
|                       |                       |          Processing   |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  Common Event |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  Common Event |
|                       |                       |          Header       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |    -  Technology      |
|                       |                       |       Independent     |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  ‘Fault       |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Heartbeat’  |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Measurement |
|                       |                       | s                     |
|                       |                       |          For Vf       |
|                       |                       |          Scaling’     |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Other’      |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘State       |
|                       |                       |          Change’      |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Syslog’     |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Threshold   |
|                       |                       |          Crossing     |
|                       |                       |          Alert’       |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |    -  Technology      |
|                       |                       |       Specify         |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  ‘Mobile      |
|                       |                       |          Flow’ Domain |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Sip         |
|                       |                       |          Signaling’   |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       |       -  ‘Voice       |
|                       |                       |          Quality’     |
|                       |                       |          Domain       |
|                       |                       |          Datatypes    |
|                       |                       |                       |
|                       |                       | -  Section 6.1.3:     |
|                       |                       |    Commands Toward    |
|                       |                       |    Event Source       |
|                       |                       |    Clients: Added a   |
|                       |                       |    statement: “Note:  |
|                       |                       |    Vendors are not    |
|                       |                       |    currently required |
|                       |                       |    to implement       |
|                       |                       |    support for        |
|                       |                       |    command            |
|                       |                       |    processing; in     |
|                       |                       |    addition, command  |
|                       |                       |    processing may be  |
|                       |                       |    supported by an    |
|                       |                       |    App-C interface in |
|                       |                       |    future.”           |
+-----------------------+-----------------------+-----------------------+
| 6/22/2017             | v5.3                  | -  JSON Schema:       |
|                       |                       |    created v28.3 by   |
|                       |                       |    correcting an      |
|                       |                       |    error in the       |
|                       |                       |    sipSignalingFields |
|                       |                       | :                     |
|                       |                       |    changed            |
|                       |                       |    vnfVendorNameField |
|                       |                       | s                     |
|                       |                       |    to                 |
|                       |                       |    vendorVnfNameField |
|                       |                       | s.                    |
|                       |                       |    Embedded the new   |
|                       |                       |    schema at the top  |
|                       |                       |    of section 4.      |
+-----------------------+-----------------------+-----------------------+
| 9/12/2017             | v5.4                  | -  Note: There no     |
|                       |                       |    changes to any     |
|                       |                       |    data structures or |
|                       |                       |    operations in this |
|                       |                       |    version.           |
|                       |                       |                       |
|                       |                       | -  JSON Schema:       |
|                       |                       |    created v28.4      |
|                       |                       |    embedded at the    |
|                       |                       |    top of section 4:  |
|                       |                       |                       |
|                       |                       |    -  Added a         |
|                       |                       |       reference to    |
|                       |                       |       eventList in    |
|                       |                       |       the properties  |
|                       |                       |       defined under   |
|                       |                       |       the schema      |
|                       |                       |       title. This     |
|                       |                       |       enables the     |
|                       |                       |       schema to       |
|                       |                       |       correctly       |
|                       |                       |       validate event  |
|                       |                       |       batches in      |
|                       |                       |       addition to     |
|                       |                       |       just events.    |
|                       |                       |                       |
|                       |                       |    -  Moved the       |
|                       |                       |       schema title to |
|                       |                       |       the top of the  |
|                       |                       |       schema and      |
|                       |                       |       changed the     |
|                       |                       |       text from       |
|                       |                       |       “Event          |
|                       |                       |       Listener” to    |
|                       |                       |       “VES Event      |
|                       |                       |       Listener”       |
|                       |                       |                       |
|                       |                       |    -  Added a schema  |
|                       |                       |       header block    |
|                       |                       |       under the title |
|                       |                       |       to clearly      |
|                       |                       |       communicate the |
|                       |                       |       schema version, |
|                       |                       |       associated API  |
|                       |                       |       and             |
|                       |                       |       last-modified   |
|                       |                       |       information     |
|                       |                       |                       |
|                       |                       | -  Changed the date   |
|                       |                       |    in the copyright   |
|                       |                       |    notice to 2017     |
+-----------------------+-----------------------+-----------------------+
| 9/19/2017             | v5.4.1                | -  Note: There no     |
|                       |                       |    changes to any     |
|                       |                       |    data structures or |
|                       |                       |    operations in this |
|                       |                       |    version.           |
|                       |                       |                       |
|                       |                       | -  Back of Cover      |
|                       |                       |    Page: updated the  |
|                       |                       |    license and        |
|                       |                       |    copyright notice   |
|                       |                       |    to comply with     |
|                       |                       |    ONAP guidelines    |
|                       |                       |                       |
|                       |                       | -  JSON Schema:       |
|                       |                       |    updated the JSON   |
|                       |                       |    schema to v28.4.1: |
|                       |                       |    updated the        |
|                       |                       |    copyright notice   |
|                       |                       |    and license to     |
|                       |                       |    comply with ONAP   |
|                       |                       |    guidelines         |
+-----------------------+-----------------------+-----------------------+
| 6/28/2018             | v6.0                  | -  Added contributors |
|                       |                       |    to the title page. |
|                       |                       |                       |
|                       |                       | -  Updated references |
|                       |                       |    to ‘vnf’ ‘vnfc’ to |
|                       |                       |    either ‘nf’ and    |
|                       |                       |    ‘nfc’ or ‘xNf’ and |
|                       |                       |    ‘xNfc’ to          |
|                       |                       |    generalize support |
|                       |                       |    across both vnfs   |
|                       |                       |    and pnfs.          |
|                       |                       |                       |
|                       |                       | -  Section 1:         |
|                       |                       |                       |
|                       |                       |    -  clarified the   |
|                       |                       |       meaning of the  |
|                       |                       |       VES acronym     |
|                       |                       |                       |
|                       |                       |    -  changed         |
|                       |                       |       references from |
|                       |                       |       ASDC to SDC and |
|                       |                       |       from MSO to SO  |
|                       |                       |                       |
|                       |                       |    -  clarified the   |
|                       |                       |       requirements    |
|                       |                       |       for eventNames. |
|                       |                       |                       |
|                       |                       |    -  Added a section |
|                       |                       |       of EventId use  |
|                       |                       |       case examples   |
|                       |                       |                       |
|                       |                       |    -  Added a new     |
|                       |                       |       section on      |
|                       |                       |       measurement     |
|                       |                       |       expansion       |
|                       |                       |       fields          |
|                       |                       |                       |
|                       |                       |    -  Added a new     |
|                       |                       |       section of      |
|                       |                       |       syslogs         |
|                       |                       |                       |
|                       |                       |    -  clarified the   |
|                       |                       |       versioning      |
|                       |                       |       section and     |
|                       |                       |       referenced the  |
|                       |                       |       new API         |
|                       |                       |       Versioning      |
|                       |                       |       section in      |
|                       |                       |       section 6.      |
|                       |                       |                       |
|                       |                       |    -  Added a list of |
|                       |                       |       all the latest  |
|                       |                       |       field block     |
|                       |                       |       version numbers |
|                       |                       |       in this version |
|                       |                       |       of the API      |
|                       |                       |       spec.           |
|                       |                       |                       |
|                       |                       | -  Section 2: updated |
|                       |                       |    the sample to show |
|                       |                       |    use of new HTTP    |
|                       |                       |    versioning         |
|                       |                       |    headers. Added a   |
|                       |                       |    note indicating    |
|                       |                       |    that support for   |
|                       |                       |    mutual SSL would   |
|                       |                       |    be provided in     |
|                       |                       |    future.            |
|                       |                       |                       |
|                       |                       | -  Section 3: updated |
|                       |                       |    the resource       |
|                       |                       |    structure remove   |
|                       |                       |    the                |
|                       |                       |    clientThrottlingSt |
|                       |                       | ate                   |
|                       |                       |    resource.          |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |    hashMaps. Changed  |
|                       |                       |    all name-value     |
|                       |                       |    pair structures to |
|                       |                       |    hashMaps causing   |
|                       |                       |    the following data |
|                       |                       |    model and JSON     |
|                       |                       |    schema (to v29.0)  |
|                       |                       |    changes:           |
|                       |                       |                       |
|                       |                       |    -  4.1.1: Common   |
|                       |                       |       Event           |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  removed      |
|                       |                       |          "field" and  |
|                       |                       |          added        |
|                       |                       |          “hashMap”    |
|                       |                       |                       |
|                       |                       |       -  removed      |
|                       |                       |          “namedArrayO |
|                       |                       | fFields”              |
|                       |                       |          and added    |
|                       |                       |          “namedHashMa |
|                       |                       | p”                    |
|                       |                       |                       |
|                       |                       |       -  added        |
|                       |                       |          arrayOfNamed |
|                       |                       | HashMap               |
|                       |                       |                       |
|                       |                       |       -  added        |
|                       |                       |          arrayOfJsonO |
|                       |                       | bject                 |
|                       |                       |                       |
|                       |                       |    -  4.2.1: Fault    |
|                       |                       |       Domain          |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          faultFields  |
|                       |                       |          version to   |
|                       |                       |          3.0 (major   |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          faultFields. |
|                       |                       | alarmAdditionalInform |
|                       |                       | ation                 |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |    -  4.2.2:          |
|                       |                       |       Heartbeat       |
|                       |                       |       Domain          |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          heartbeatFie |
|                       |                       | ldsVersion            |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          heartbeatFie |
|                       |                       | lds.additionalFields  |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |    -  4.2.3:          |
|                       |                       |       Measurement     |
|                       |                       |       Domain          |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          measurementF |
|                       |                       | ieldsVersion          |
|                       |                       |          to 3.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          measurementF |
|                       |                       | ields.additionalField |
|                       |                       | s                     |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          measurement. |
|                       |                       | additionalMesurements |
|                       |                       |          to reference |
|                       |                       |          a            |
|                       |                       |          namedHashMap |
|                       |                       |          [ ]          |
|                       |                       |                       |
|                       |                       |       -  modified     |
|                       |                       |          measurementF |
|                       |                       | ields.featureUsageArr |
|                       |                       | ay                    |
|                       |                       |          to reference |
|                       |                       |          a hashmap    |
|                       |                       |          and removed  |
|                       |                       |          ‘featuresInU |
|                       |                       | se’                   |
|                       |                       |                       |
|                       |                       |       -  added the    |
|                       |                       |          following    |
|                       |                       |          datatypes    |
|                       |                       |          which are    |
|                       |                       |          now          |
|                       |                       |          referenced   |
|                       |                       |          as items in  |
|                       |                       |          arrays       |
|                       |                       |          within       |
|                       |                       |          measurementF |
|                       |                       | ields:                |
|                       |                       |          hugePages,   |
|                       |                       |          load,        |
|                       |                       |          machineCheck |
|                       |                       | Exception,            |
|                       |                       |          processStats |
|                       |                       |                       |
|                       |                       |    -  4.2.5: Other    |
|                       |                       |       Domain          |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  Change the   |
|                       |                       |          otherFieldsV |
|                       |                       | ersion                |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          otherFields. |
|                       |                       | nameValuePairs        |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |          and renamed  |
|                       |                       |          it hashMap   |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          otherFields. |
|                       |                       | hashOfNameValuePairAr |
|                       |                       | rrays                 |
|                       |                       |          to reference |
|                       |                       |          a            |
|                       |                       |          namedHashMap |
|                       |                       |          and renamed  |
|                       |                       |          it           |
|                       |                       |          arrayOfNamed |
|                       |                       | HashMap               |
|                       |                       |                       |
|                       |                       |    -  4.2.7: State    |
|                       |                       |       Change Domain   |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          stateChangeF |
|                       |                       | iledsVersion          |
|                       |                       |          to 3.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          stateChangeF |
|                       |                       | ields.additionalField |
|                       |                       | s                     |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |    -  4.2.9:          |
|                       |                       |       Threshold       |
|                       |                       |       Crossing Alert  |
|                       |                       |       Domain          |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          thresholdCro |
|                       |                       | ssingAlertFieldsVersi |
|                       |                       | on                    |
|                       |                       |          to 3.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          thresholdCro |
|                       |                       | ssingAlertFields.addi |
|                       |                       | tionalFields          |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |       -  counter:     |
|                       |                       |          removed name |
|                       |                       |          and value    |
|                       |                       |          elements and |
|                       |                       |          replaced     |
|                       |                       |          with a       |
|                       |                       |          hashMap      |
|                       |                       |                       |
|                       |                       |    -  4.3.1: Mobile   |
|                       |                       |       Flow Domain     |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          mobileFlowFi |
|                       |                       | eldsVersion           |
|                       |                       |          to 3.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          mobileFlowFi |
|                       |                       | elds.additionalFields |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |       -  gtpPerFlowMe |
|                       |                       | trics:                |
|                       |                       |          modified     |
|                       |                       |          ipTosCountLi |
|                       |                       | st                    |
|                       |                       |          to reference |
|                       |                       |          hashmap      |
|                       |                       |                       |
|                       |                       |       -  gtpPerFlowMe |
|                       |                       | trics:                |
|                       |                       |          modified     |
|                       |                       |          mobileQciCos |
|                       |                       | CountList             |
|                       |                       |          to reference |
|                       |                       |          hashmap      |
|                       |                       |                       |
|                       |                       |       -  gtpPerFlowMe |
|                       |                       | trics:                |
|                       |                       |          modified     |
|                       |                       |          tcpFlagCount |
|                       |                       | List                  |
|                       |                       |          to reference |
|                       |                       |          hashmap      |
|                       |                       |                       |
|                       |                       |    -  4.3.2: Sip      |
|                       |                       |       Signaling       |
|                       |                       |       Domain          |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  changed the  |
|                       |                       |          sigSignaling |
|                       |                       | FieldsVersion         |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          sipSignaling |
|                       |                       | Fields.additionalInfo |
|                       |                       | rmation               |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       |    -  4.3.3: Voice    |
|                       |                       |       Quality Domain  |
|                       |                       |       Datatypes:      |
|                       |                       |                       |
|                       |                       |       -  change the   |
|                       |                       |          voiceQuality |
|                       |                       | FieldsVersion         |
|                       |                       |          to 2.0       |
|                       |                       |          (major       |
|                       |                       |          change)      |
|                       |                       |                       |
|                       |                       |       -  changed      |
|                       |                       |          voiceQuality |
|                       |                       | Fields.additionalInfo |
|                       |                       | rmation               |
|                       |                       |          to reference |
|                       |                       |          a hashMap    |
|                       |                       |                       |
|                       |                       | -  Section 4: added   |
|                       |                       |    notes at the top   |
|                       |                       |    of section 4       |
|                       |                       |    clarifying         |
|                       |                       |    expectations and   |
|                       |                       |    requirements for   |
|                       |                       |    optional fields,   |
|                       |                       |    extensible fields  |
|                       |                       |    and keys sent      |
|                       |                       |    through extensible |
|                       |                       |    fields.            |
|                       |                       |                       |
|                       |                       | -  Common Event Data  |
|                       |                       |    Types: Section     |
|                       |                       |    4.1.1.9 Changed    |
|                       |                       |    vendorVnfNameField |
|                       |                       | s                     |
|                       |                       |    to                 |
|                       |                       |    vendorNfNameFields |
|                       |                       | ;                     |
|                       |                       |    updated Section    |
|                       |                       |    4.3.2 SipSignaling |
|                       |                       |    and 4.3.3 Voice    |
|                       |                       |    Quality to refer   |
|                       |                       |    to the renamed     |
|                       |                       |    object             |
|                       |                       |                       |
|                       |                       | -  Common Event       |
|                       |                       |    Header Section     |
|                       |                       |    4.1.2:             |
|                       |                       |                       |
|                       |                       |    -  clarified the   |
|                       |                       |       descriptions of |
|                       |                       |       eventId,        |
|                       |                       |       reportingEntity |
|                       |                       | Name,                 |
|                       |                       |       sourceName and  |
|                       |                       |       startEpochMicro |
|                       |                       | seconds.              |
|                       |                       |                       |
|                       |                       |    -  Added           |
|                       |                       |       ‘notification’  |
|                       |                       |       and             |
|                       |                       |       ‘pngRegistratio |
|                       |                       | n’                    |
|                       |                       |       to the domain   |
|                       |                       |       enumeration.    |
|                       |                       |                       |
|                       |                       |    -  added a new     |
|                       |                       |       timeZoneOffsest |
|                       |                       |       field           |
|                       |                       |                       |
|                       |                       | -  Fault Domain       |
|                       |                       |    Section 4.2.1:     |
|                       |                       |    clarified the      |
|                       |                       |    definitions of     |
|                       |                       |    alarmCondition,    |
|                       |                       |    eventSeverity and  |
|                       |                       |    specificProblem    |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3: changed the |
|                       |                       |    name of this       |
|                       |                       |    domain from        |
|                       |                       |    ‘measurementsForVf |
|                       |                       | Scaling’              |
|                       |                       |    to ‘measurement’   |
|                       |                       |                       |
|                       |                       |    -  measurementsFor |
|                       |                       | VfScaling             |
|                       |                       |       measurement     |
|                       |                       |                       |
|                       |                       |    -  measurementsFor |
|                       |                       | VfScalingFields       |
|                       |                       |       measurementFiel |
|                       |                       | ds                    |
|                       |                       |                       |
|                       |                       |    -  measurementsFor |
|                       |                       | VfScalingVersion      |
|                       |                       |       measurementFiel |
|                       |                       | dsVersion             |
|                       |                       |                       |
|                       |                       |    -  the ‘mfvs’      |
|                       |                       |       abbreviation    |
|                       |                       |       measurement     |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3 cpuUsage:    |
|                       |                       |    added seven        |
|                       |                       |    optional fields to |
|                       |                       |    this structure:    |
|                       |                       |    cpuCapacityContent |
|                       |                       | ion,                  |
|                       |                       |    cpuDemandAvg,      |
|                       |                       |    cpuDemandMhz,      |
|                       |                       |    cpuDemandPct,      |
|                       |                       |    cpuLatencyAverage, |
|                       |                       |    cpuOverheadAvg,    |
|                       |                       |    cpuSwapWaitTime    |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3 diskUsage:   |
|                       |                       |    added ten optional |
|                       |                       |    fields to this     |
|                       |                       |    structure:         |
|                       |                       |    diskBusResets,     |
|                       |                       |    diskCommandsAborte |
|                       |                       | d,                    |
|                       |                       |    diskCommandsAvg,   |
|                       |                       |    diskFlushRequests, |
|                       |                       |    diskFlushTime,     |
|                       |                       |    diskReadCommandsAv |
|                       |                       | g,                    |
|                       |                       |    diskTime,          |
|                       |                       |    diskTotalReadLaten |
|                       |                       | cyAvg,                |
|                       |                       |    diskTotalWriteLate |
|                       |                       | ncyAvg,               |
|                       |                       |    diskWriteCommandsA |
|                       |                       | vg                    |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3: added a new |
|                       |                       |    ‘ipmi’ datatype    |
|                       |                       |    along with         |
|                       |                       |    following          |
|                       |                       |    ‘supporting’       |
|                       |                       |    datatypes:         |
|                       |                       |    ipmiBaseboardTempe |
|                       |                       | rature,               |
|                       |                       |    ipmiBaseboardVolta |
|                       |                       | geRegulator,          |
|                       |                       |    ipmiBattery,       |
|                       |                       |    ipmiFan,           |
|                       |                       |    ipmiGlobalAggregat |
|                       |                       | eTemperatureMargin,   |
|                       |                       |    ipmiHsbp, ipmiNic, |
|                       |                       |    ipmiPowerSupply,   |
|                       |                       |    ipmiProcessor,     |
|                       |                       |    processorDimmAggre |
|                       |                       | gateThermalMargin     |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3: added a new |
|                       |                       |    ‘load’ datatype    |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3 memoryUsage: |
|                       |                       |    added eight        |
|                       |                       |    optional fields to |
|                       |                       |    this structure:    |
|                       |                       |    memoryDemand,      |
|                       |                       |    memoryLatencyAvg,  |
|                       |                       |    memorySharedAvg,   |
|                       |                       |    memorySwapInAvg,   |
|                       |                       |    memorySwapInRateAv |
|                       |                       | g,                    |
|                       |                       |    memorySwapOutAvg,  |
|                       |                       |    memorySwapOutRateA |
|                       |                       | vg,                   |
|                       |                       |    memorySwapUsedAvg  |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3: modified    |
|                       |                       |    measurementFields  |
|                       |                       |    to include the     |
|                       |                       |    following new      |
|                       |                       |    fields:            |
|                       |                       |    hugePagesArray,    |
|                       |                       |    ipmi, loadArray,   |
|                       |                       |    memoryErrors,      |
|                       |                       |    processStatusArray |
|                       |                       | ,                     |
|                       |                       |    rdtArray           |
|                       |                       |                       |
|                       |                       | -  Measurements       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.3 renamed      |
|                       |                       |    vNicPerformance to |
|                       |                       |    nicPerformance and |
|                       |                       |    changed            |
|                       |                       |    vNicIdentifer to   |
|                       |                       |    nicIdentifier      |
|                       |                       |                       |
|                       |                       | -  Notification       |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.4: added       |
|                       |                       |    notificationFields |
|                       |                       |    to support a new   |
|                       |                       |    notification       |
|                       |                       |    domain.            |
|                       |                       |                       |
|                       |                       | -  pnfRegistration    |
|                       |                       |    Domain Section     |
|                       |                       |    4.2.7: added       |
|                       |                       |    pnfRegistrationFie |
|                       |                       | lds                   |
|                       |                       |    to support a new   |
|                       |                       |    registration       |
|                       |                       |    domain.            |
|                       |                       |                       |
|                       |                       | -  sysLog Domain      |
|                       |                       |    Section 4.2.8:     |
|                       |                       |    added two new      |
|                       |                       |    fields:            |
|                       |                       |    syslogMsgHost and  |
|                       |                       |    syslogTs.          |
|                       |                       |    Clarified field    |
|                       |                       |    descriptions.      |
|                       |                       |    Clarified          |
|                       |                       |    syslogSData        |
|                       |                       |    example.           |
|                       |                       |                       |
|                       |                       | -  endOfCallVqmSummar |
|                       |                       | ies                   |
|                       |                       |    Section 4.3.3.1:   |
|                       |                       |                       |
|                       |                       |    -  converted       |
|                       |                       |       endpointJitter  |
|                       |                       |       into two        |
|                       |                       |       fields:         |
|                       |                       |       endpointAverage |
|                       |                       | Jitter                |
|                       |                       |       and             |
|                       |                       |       endpointMaxJitt |
|                       |                       | er                    |
|                       |                       |                       |
|                       |                       |    -  converted       |
|                       |                       |       localJitter     |
|                       |                       |       into two        |
|                       |                       |       fields:         |
|                       |                       |       localAverageJit |
|                       |                       | ter                   |
|                       |                       |       and             |
|                       |                       |       localMaxJitter  |
|                       |                       |                       |
|                       |                       |    -  added two       |
|                       |                       |       fields:         |
|                       |                       |       localAverageJit |
|                       |                       | terBufferDelay        |
|                       |                       |       and             |
|                       |                       |       localMaxJitterB |
|                       |                       | ufferDelay            |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |       endpointRtpOcte |
|                       |                       | tsLost                |
|                       |                       |       and             |
|                       |                       |       endpointRtpPack |
|                       |                       | etsLost               |
|                       |                       |                       |
|                       |                       |    -  added           |
|                       |                       |       localRtpOctetsL |
|                       |                       | ost                   |
|                       |                       |       and             |
|                       |                       |       localRtpPackets |
|                       |                       | Lost                  |
|                       |                       |                       |
|                       |                       |    -  converted       |
|                       |                       |       packetsLost     |
|                       |                       |       into            |
|                       |                       |       oneWayDelay     |
|                       |                       |                       |
|                       |                       | -  API Versioning:    |
|                       |                       |                       |
|                       |                       |    -  Section 1.4:    |
|                       |                       |       clarified the   |
|                       |                       |       versioning      |
|                       |                       |       section and     |
|                       |                       |       linked it to    |
|                       |                       |       the following   |
|                       |                       |       new section     |
|                       |                       |       6.1.2           |
|                       |                       |                       |
|                       |                       |    -  Section 6.1.2:  |
|                       |                       |       Added           |
|                       |                       |       requirements    |
|                       |                       |       for HTTP        |
|                       |                       |       headers         |
|                       |                       |       communicating   |
|                       |                       |       minor, patch    |
|                       |                       |       and latest      |
|                       |                       |       version         |
|                       |                       |       information.    |
|                       |                       |                       |
|                       |                       |    -  Section 2 and 6 |
|                       |                       |       sample          |
|                       |                       |       messages:       |
|                       |                       |       clarified       |
|                       |                       |       examples to use |
|                       |                       |       the new HTTP    |
|                       |                       |       headers         |
|                       |                       |                       |
|                       |                       | -  Section 6.1.4:     |
|                       |                       |    Added a section    |
|                       |                       |    specifying message |
|                       |                       |    size limits.       |
|                       |                       |                       |
|                       |                       | -  Section2 6.2.6.1   |
|                       |                       |    and 6.3.6.1:       |
|                       |                       |    corrected          |
|                       |                       |    additionalInformat |
|                       |                       | ion                   |
|                       |                       |    examples to use    |
|                       |                       |    hashMap instead of |
|                       |                       |    name-value pair    |
|                       |                       |    fields.            |
|                       |                       |                       |
|                       |                       | -  Section 7: Added a |
|                       |                       |    section on         |
|                       |                       |    Terminology.       |
|                       |                       |                       |
|                       |                       | -  Command List       |
|                       |                       |    Processing:        |
|                       |                       |    removed command    |
|                       |                       |    list processing    |
|                       |                       |    from the document  |
|                       |                       |    and schema:        |
|                       |                       |                       |
|                       |                       |    -  Modified the    |
|                       |                       |       Section 3       |
|                       |                       |       resource        |
|                       |                       |       structure to    |
|                       |                       |       align with      |
|                       |                       |       these changes.  |
|                       |                       |                       |
|                       |                       |    -  Removed Section |
|                       |                       |       4 Datatypes:    |
|                       |                       |       command,        |
|                       |                       |       commandList,    |
|                       |                       |       eventDomainThro |
|                       |                       | ttleSpecification,    |
|                       |                       |       eventDomainThro |
|                       |                       | ttleSpecificationList |
|                       |                       | ,                     |
|                       |                       |       eventThrottling |
|                       |                       | State,                |
|                       |                       |       suppressedNvPai |
|                       |                       | rs                    |
|                       |                       |                       |
|                       |                       |    -  Removed Section |
|                       |                       |       6.1 description |
|                       |                       |       of commands     |
|                       |                       |       toward event    |
|                       |                       |       source clients  |
|                       |                       |                       |
|                       |                       | -  Removed Section    |
|                       |                       |    6.4 operation:     |
|                       |                       |    provideThrottlingS |
|                       |                       | tate                  |
+-----------------------+-----------------------+-----------------------+
| 7/30/2018             | v7.0                  | -  General:           |
|                       |                       |                       |
|                       |                       |    -  Fixed typos     |
|                       |                       |       throughout      |
|                       |                       |                       |
|                       |                       |    -  Changed example |
|                       |                       |       versions to v7  |
|                       |                       |                       |
|                       |                       | -  Section1:          |
|                       |                       |                       |
|                       |                       |    -  Clarified       |
|                       |                       |       casing and use  |
|                       |                       |       of dashes       |
|                       |                       |       versus colons   |
|                       |                       |       in eventName    |
|                       |                       |       examples        |
|                       |                       |                       |
|                       |                       |    -  Updated all     |
|                       |                       |       field block     |
|                       |                       |       versions        |
|                       |                       |                       |
|                       |                       | -  Section 2: added a |
|                       |                       |    note clarifying    |
|                       |                       |    that TLS 1.2 or    |
|                       |                       |    higher must be     |
|                       |                       |    used for HTTPS     |
|                       |                       |    connections.       |
|                       |                       |                       |
|                       |                       | -  Section 4 embedded |
|                       |                       |    schema changed to  |
|                       |                       |    v30:               |
|                       |                       |                       |
|                       |                       |    -  Added “         |
|                       |                       |       ‘additionalProp |
|                       |                       | erties’:              |
|                       |                       |       false ” to      |
|                       |                       |       objects to      |
|                       |                       |       reject events   |
|                       |                       |       that attempt to |
|                       |                       |       send properties |
|                       |                       |       that are not    |
|                       |                       |       listed in the   |
|                       |                       |       ‘properties’    |
|                       |                       |       keyword. Note:  |
|                       |                       |       does not affect |
|                       |                       |       hashmap         |
|                       |                       |       extensible      |
|                       |                       |       fields.         |
|                       |                       |                       |
|                       |                       |    -  Changed all     |
|                       |                       |       versions in all |
|                       |                       |       field blocks    |
|                       |                       |       from number to  |
|                       |                       |       string enum     |
|                       |                       |       with the        |
|                       |                       |       version number  |
|                       |                       |       fixed by the    |
|                       |                       |       enum so the     |
|                       |                       |       schema can      |
|                       |                       |       validate events |
|                       |                       |       that attempt to |
|                       |                       |       send            |
|                       |                       |       non-standard    |
|                       |                       |       field blocks.   |
|                       |                       |                       |
|                       |                       |    -  Changed syslog  |
|                       |                       |       additionalField |
|                       |                       | s                     |
|                       |                       |       to a hashMap    |
|                       |                       |                       |
|                       |                       | -  Section 4:         |
|                       |                       |                       |
|                       |                       |    -  Fixed section   |
|                       |                       |       heading numbers |
|                       |                       |       that were the   |
|                       |                       |       same.           |
|                       |                       |                       |
|                       |                       |    -  4.1.1:          |
|                       |                       |       jsonObjectInsta |
|                       |                       | nce:                  |
|                       |                       |       added an        |
|                       |                       |       optional        |
|                       |                       |       recursive       |
|                       |                       |       jsonObject and  |
|                       |                       |       removed all     |
|                       |                       |       required fields |
|                       |                       |       from this       |
|                       |                       |       object          |
|                       |                       |                       |
|                       |                       |    -  4.1.2:          |
|                       |                       |       commonEventHead |
|                       |                       | er:                   |
|                       |                       |                       |
|                       |                       |       -  nfVendorName |
|                       |                       | :                     |
|                       |                       |          added this   |
|                       |                       |          optional     |
|                       |                       |          field        |
|                       |                       |                       |
|                       |                       |       -  timeZoneOffs |
|                       |                       | et:                   |
|                       |                       |          changed from |
|                       |                       |          number to    |
|                       |                       |          string with  |
|                       |                       |          a particular |
|                       |                       |          format       |
|                       |                       |          specified    |
|                       |                       |                       |
|                       |                       |       -  version was  |
|                       |                       |          changed from |
|                       |                       |          number to    |
|                       |                       |          string (as   |
|                       |                       |          were all the |
|                       |                       |          version      |
|                       |                       |          fields of    |
|                       |                       |          all the      |
|                       |                       |          field        |
|                       |                       |          blocks)      |
|                       |                       |                       |
|                       |                       |       -  vesCommonEve |
|                       |                       | ntListenerVersion:    |
|                       |                       |          added this   |
|                       |                       |          required     |
|                       |                       |          field as a   |
|                       |                       |          string       |
|                       |                       |          enumeration  |
|                       |                       |                       |
|                       |                       |    -  4.2.3:          |
|                       |                       |       Measurements    |
|                       |                       |       Domain:         |
|                       |                       |                       |
|                       |                       |       -  Added a note |
|                       |                       |          clarifying   |
|                       |                       |          that NFs are |
|                       |                       |          required to  |
|                       |                       |          report       |
|                       |                       |          exactly one  |
|                       |                       |          Measurement  |
|                       |                       |          event per    |
|                       |                       |          period per   |
|                       |                       |          sourceName   |
|                       |                       |                       |
|                       |                       |       -  diskUsage:   |
|                       |                       |          added four   |
|                       |                       |          new optional |
|                       |                       |          fields:      |
|                       |                       |          diskWeighted |
|                       |                       | IoTimeAve,            |
|                       |                       |          diskWeighted |
|                       |                       | IoTimeLast,           |
|                       |                       |          diskWeighted |
|                       |                       | IoTimeMax,            |
|                       |                       |          diskWeighted |
|                       |                       | IoTimeMin             |
|                       |                       |                       |
|                       |                       |       -  memoryUsage: |
|                       |                       |          add one new  |
|                       |                       |          optional     |
|                       |                       |          field:       |
|                       |                       |          percentMemor |
|                       |                       | yUsage                |
|                       |                       |                       |
|                       |                       |       -  nicPerforman |
|                       |                       | ce:                   |
|                       |                       |          added nine   |
|                       |                       |          new optional |
|                       |                       |          fields:      |
|                       |                       |          administrati |
|                       |                       | veState,              |
|                       |                       |          operationalS |
|                       |                       | tate,                 |
|                       |                       |          receivedPerc |
|                       |                       | entDiscard,           |
|                       |                       |          receivedPerc |
|                       |                       | entError,             |
|                       |                       |          receivedUtil |
|                       |                       | ization,              |
|                       |                       |          speed,       |
|                       |                       |          transmittedP |
|                       |                       | ercentDiscard,        |
|                       |                       |          transmittedP |
|                       |                       | ercentError,          |
|                       |                       |          transmittedU |
|                       |                       | tilization            |
|                       |                       |                       |
|                       |                       |       -  processorDim |
|                       |                       | mAggregateThermalMarg |
|                       |                       | in:                   |
|                       |                       |          make the     |
|                       |                       |          thermalMargi |
|                       |                       | n                     |
|                       |                       |          field        |
|                       |                       |          required     |
|                       |                       |                       |
|                       |                       |    -  4.2.8: Syslog   |
|                       |                       |       Domain:         |
|                       |                       |                       |
|                       |                       | -  Corrected the      |
|                       |                       |    example at the end |
|                       |                       |    of the section     |
+-----------------------+-----------------------+-----------------------+

.. |image0| image:: media/image1.png
   :width: 6.48926in
   :height: 4.86694in
.. |image1| image:: media/image2.png
   :width: 6.5in
   :height: 4.8745in
.. |image2| image:: media/image3.png
   :width: 3.76033in
   :height: 1.16677in
.. |image3| image:: media/image5.png
   :width: 4.75347in
   :height: 2.57361in
.. |image4| image:: media/image6.png
   :width: 4.74722in
   :height: 2.56667in
