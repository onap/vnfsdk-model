.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017 AT&T Intellectual Property, All rights reserved
.. Copyright 2017-2018 Huawei Technologies Co., Ltd.

==========================
VES Event Registration 2.0
==========================

.. contents:: Table of Contents

Introduction
============

This document specifies a YAML format for the registration of VES
Events. The YAML format enables both human designers and applications to
parse and understand the fields that will be sent by event sources in
conjunction with specific types of events, which are identified by their
eventNames.

The semantics of the YAML format are easily extensible to accommodate
processing needs that may arise in the future. Among the types of
information specified in the YAML are field optionality, restrictions on
field values, and event handling recommendations and requirements.

This document should be read in conjunction with the VES Event Listener
service specification, which defines the Common Event Format and
introduces the concept of specific types of events, identified by
eventNames.

Audience
--------

This document is intended to support the following groups:

-  VNF Vendors

-  Service Provider (e.g., AT&T) Teams responsible for deploying VNFs
   within their infrastructure

VNF vendors will provide a YAML file to the Service Provider that
describes the events that their VNFs generate. Using the semantics and
syntax supported by YAML, vendors will indicate specific conditions that
may arise, and recommend actions that should be taken at specific
thresholds, or if specific conditions repeat within a specified time
interval.

Based on the vendor’s recommendations, the Service Provider may create
another YAML, which finalizes their engineering rules for the processing
of the vendor’s events. The Service Provider may alter the threshold
levels recommended by the vendor, and may modify and more clearly
specify actions that should be taken when specified conditions arise.
The Service Provided-created version of the YAML will be distributed to
Service Provider applications at design time.

Goal
----

The goal of the YAML is to completely describe the processing of VNF
events in a way that can be compiled or interpreted by applications
across a Service Provider’s infrastructure.

Relation to the Common Event Format
-----------------------------------

The Common Event Format described in the VES Event Listener service
specification defines the structure of VES events including optional
fields that may be provided.

Specific eventNames registered by the YAML (e.g., an InvalidLicense
fault), may require that certain fields, which are optional in the
Common Event Format, be present when events with that eventName are
published. For example, a fault eventName which communicates an
‘InvalidLicense’ condition, may be registered to require that the
configured ‘licenseKey’ be provided as a name-value pair in the Common
Event Format’s ‘additionalFields’ structure, within the ‘faultFields’
block. Anytime an ‘InvalidLicense’ fault event is detected, designers,
applications and microservices across the Service Provider’s
infrastructure can count on that name-value pair being present.

The YAML registration may also restrict ranges or enumerations defined
in the Common Event Format. For example, eventSeverity is an enumerated
string within the Common Event Format with several values ranging from
‘NORMAL’ to ‘CRITICAL’. The YAML registration for a particular eventName
may require that it always be sent with eventSeverity set to a single
value (e.g., ‘MINOR’), or to a subset of the possible enumerated values
allowed by the Common Event Format (e.g., ‘MINOR’ or ‘NORMAL’).

Relation to Service Design and Creation
---------------------------------------

Event registration for a VNF (or other event source) is provided to the
Service Provider’s Service Creation and Design Environment (e.g., SDC)
as a set of two YAML files consisting of the vendor recommendation YAML
and (optionally) the final Service Provider YAML. These YAML files
describe all the eventNames that that VNF (or other event source)
generates.

Once their events are registered, the Service Creation and Design
Environment can then list the registered eventNames (e.g., as a drop
down list), for each VNF or other event source (e.g., a service), and
enable designers to study the YAML registrations for specific
eventNames. YAML registrations are both human readable and machine
readable.

The final Service Provider YAML is a type of Service Design and Creation
‘artifact’, which can be distributed to Service Provider applications at
design time: notably, to applications involved in the collection and
processing of VNF events. It can be parsed by those applications so they
can support the receipt and processing of VNF events, without the need
for any manual, VNF-specific development.

YAML Files
==========

YAML Specification Conformance
------------------------------

YAML files should conform to version 1.2 of the YAML specification
available at: http://yaml.org/spec/1.2/spec.html.

Filename
--------

YAML file names should conform to the following naming convention:

   {NamingCode}_{ModelType}_{v#}_{AdditionalInfo}.yml

The NamingCode identifies the entity, whose events are being registered
in the yaml, with a naming code that was established in the Service
Providers’ Service Design and Creation Environment (SDC). Example Naming
codes are:

-  tbcx

-  sgsn-mme

The ModelType describes the type of entity whose events are being
registered. It consists of values like:

-  service

-  vfModule

-  vnf

-  vnfc

The ‘#’ should be replaced with the current numbered version of the
file. Note that ‘#’ can be an integer or a number of the form x.y or
x.y.z (where x is the major number, y is the minor number and z is the
patch number)

Additional descriptive info may be added after the version information

Example file name:

-  vIsbcSsc_vnfc_v1.yml

File Structure
--------------

Each eventType is registered as a distinct YAML ‘document’.

YAML files consist of a series of YAML documents delimited by ‘---‘ and
‘…’ for example:

   ---

   # Event Registration for eventName ‘name1’

   # details omitted

   ...

   ---

   # Event Registration for eventName ‘name2’

   # details omitted

   ...

   ---

   # Event Registration for eventName ‘name3’

   # details omitted

   ...

YAML Syntax and Semantics
=========================

YAML registration documents show each relevant VES Common Event Model
object and field (i.e., each element) for the eventName being
registered, including any extensible fields (e.g., specific name-value
pairs).

Qualifiers
----------

Each object or field name in the eventName being registered is followed
by a ‘qualifier’, which consists of a colon and two curly braces, for
example:

   “objectOrFieldName: { }”

The curly braces contain meta-information about that object or field
name (also known as the ‘element’), such as whether it is required to be
present, what values it may have, what handling it should trigger, etc…

Semantics have been defined for the following types of meta-information
within the curly braces:

Action
~~~~~~

The ‘action’ keyword may be applied to field values or to the event as a
whole. The ‘action’ keyword specifies a set of actions that should be
taken if a specified trigger occurs. For example, the ‘action’ keyword
may specify that a threshold crossing alert (i.e., tca) be generated,
and/or that a specific microservice handler be invoked, and/or that a
specific named-condition be asserted. In the Rules section of the YAML
file, tca’s and microservices may be defined on individual
named-conditions or on logical combinations of named-conditions.

The ‘action:’ keyword is followed by five values in square brackets. The
first two values communicate the trigger, and the last three values
communicate the actions to be taken if that trigger occurs:

1. The first value conveys the trigger level. If the field on which the
   action is defined reaches or passes through that level, then the
   trigger fires. If a specific level is not important to the
   recommended action, the ‘any’ keyword may be used as the first value.
   (Note: ‘any’ is often used when an action is defined on the ‘event’
   structure as a whole).

2. The second value indicates the direction of traversal of the level
   specified in the first value. The second value may be ‘up’, ‘down’,
   ‘at’ or ‘any’. ‘any’ is used if the direction of traversal is not
   important. ‘at’ implies that it traversed (or exactly attained) the
   trigger level but it doesn’t matter if the traversal was in the up
   direction or down direction. Note: If ‘up’, ‘down’ or ‘at’ are used,
   the implication is that the microservices processing the events
   within the service provider are maintaining state (e.g., to know that
   a measurement field traversed a trigger level in an ‘up’ direction,
   the microservice would have to know that the field was previously
   below the trigger level). When initially implementing support for
   YAML actions, a service provider may choose to use and interpret
   these keywords in a simpler way to eliminate the need to handle
   state. Specifically, they may choose to define and interpret all ‘up’
   guidance to mean ‘at the indicated trigger level or greater’, and
   they may choose to define and interpret all ‘down’ guidance to mean
   ‘at the indicated trigger level or lower’.

3. The third value optionally names the condition that has been attained
   when the triggers fires (e.g., ‘invalidLicence’ or
   ‘capacityExhaustion’). Named-conditions should be expressed in camel
   case with no underscores, hyphens or spaces and with the first letter
   in lowercase. In the Rules section of the YAML file, named-conditions
   may be used to specify events that should be generated and/or
   microservices that should be invoked. If it is not important to name
   a condition, then the string ‘null’ (without the quotes) must be used
   as the third value.

4. The fourth value recommends a specific microservice (e.g., ‘rebootVm’
   or ‘rebuildVnf’) supported by the Service Provider, be invoked if the
   trigger is attained. Design time processing of the YAML by the
   service provider can use these directives to automatically establish
   policies and configure flows that need to be in place to support the
   recommended runtime behavior.

..

   If a vendor wants to recommend an action, it can either work with the
   service provider to identify and specify microservices that the
   service provider support, or, the vendor may simply indicate and
   recommend a generic microservice function by prefixing ‘RECO-’ in
   front of the microservice name, which should be expressed in upper
   camel case with no underscores, hyphens or spaces.

   A fourth value must be provided. If not needed, the string ‘null’
   (without the quotes) must be used.

5. The fifth value indicates a specific named event (e.g., a TCA) that
   should be generated if the trigger occurs. This field must be
   provided as a VES eventName or, if not needed, as the string ‘null’
   (without the quotes).

..

   When an event is specified, a YAML registration for that eventName
   should be added to the event registrations within the YAML file.

Examples:

-  event: { action: [ any, any, null, rebootVm, null ] }

..

   # whenever the above event occurs, the VM should be rebooted

-  fieldname: { action: [ 80, up, null, null, tcaUpEventName ], action:
      [ 60, down, overcapacity, null, null ] }

..

   # when the value of fieldname crosses 80 in an up direction,
   tcaUpEventName

   should be published; if the fieldname crosses 60 in a down direction
   an

   ‘overCapacity’ named-condition is asserted.

AggregationRole
~~~~~~~~~~~~~~~

The ‘aggregationRole’ keyword is applied to a keyValuePair.

AggregationRole may be set to one of the following:

-  counter

-  index

-  reference

If needed, the aggergationRole setting tells the receiving event
processor how to aggregate the extensible keyValuePair data. Data
aggregation may use a combination of ‘index’ and ‘reference’ data fields
as aggregation keys while applying aggregation formulas, such as
summation or average on the ‘counter’ fields.

Example 1:

   Interpretation of the below: If additionalMeasurements is supplied,
   it must have key name1 and name1’s value should be interpreted as an
   index:

-  additionalMeasurements: {presence: optional, structure: {

..

   keyValuePair: {presence: required, structure: {

   key: {presence: required, value: name1},

   value: {presence: required, aggregationRole: index }

   }},

   . . .

   }}

Example 2:

-  Let’s say a vnf wants to send the following ‘TunnelTraffic’ fields
      through a VES arrayOfNamedHashMap structure (specifically through
      additionalMeasurements in the VES measurementField block):

+-------------+-------------+-------------+-------------+-------------+
| Tunnel Name | Tunnel Type | Total       | Total       | Total       |
|             |             | Output      | Output      | Output      |
|             |             | Bytes       | Packets     | Errors      |
+=============+=============+=============+=============+=============+
| ST6WA21CRS: | PRIMARY     | 2457205     | 21505       | 0           |
| TUNNEL-TE40 |             |             |             |             |
| 018         |             |             |             |             |
+-------------+-------------+-------------+-------------+-------------+
| ST6WA21CRS: | PRIMARY     | 46677       | 220         | 0           |
| TUNNEL-TE10 |             |             |             |             |
| 29          |             |             |             |             |
+-------------+-------------+-------------+-------------+-------------+
| ST6WA21CRS: | PRIMARY     | 80346       | 577         | 0           |
| TUNNEL-TE10 |             |             |             |             |
| 28          |             |             |             |             |
+-------------+-------------+-------------+-------------+-------------+

-  Tunnel Name is an index, Tunnel Type is reference data and the other
      three columns are counters

-  The first three columns would be sent through VES as follows:

additionalMeasurements: { presence: required, isHomogeneous: true,
array: [

namedHashMap: { presence: required, structure: {

name: { presence: required, value: "TunnelTraffic" },

hashMap: { presence: required, array: [

keyValuePair: { presence: required, aggregationRole: index, structure: {

key: { presence: required, value: TunnelName },

value: { presence: required }

}},

keyValuePair: { presence: required, aggregationRole: reference,
structure: {

key: { presence: required, value: TunnelType },

value: { presence: required }

}},

keyValuePair: { presence: required, aggregationRole: counter, structure:
{

key: { presence: required, value: TotalOutputBytes },

value: { presence: required, castTo: integer }

}}

]}

}}

]}

Array
~~~~~

The ‘array’ keyword indicates that the element is an array; ‘array:’ is
following by square brackets which contain the elements of the array.
Note that unlike JSON itself, the YAML registration will explicitly
declare the array elements and will not communicate them anonymously.

Examples:

-  element: { array: [

..

   firstArrayElement: { },

   secondArrayElement: { }

   ] }

CastTo
~~~~~~

The ‘castTo’ keyword is applied to ‘value’ keywords. It tells the
receiving event processor to cast the supplied value from its standard
VES datatype (typically a string) to some other datatype. If not
supplied the implication is the standard VES datatype applies.

A value may be castTo one and only one of the following data types:

-  boolean

-  integer

-  number (note: this supports decimal values as well as integral
      values)

-  string

Example:

-  fieldname: { value: [ x, y, z ], castTo: number } # only values ‘x’,
      ‘y’, or ‘z’ allowed

..

   # each must be cast to a number

-  additionalMeasurements: {presence: optional, structure: {

..

   keyValuePair: {presence: required, structure: { # if
   additionalMeasurements is

   key: {presence: required, value: name1}, # supplied, it must have key
   ‘name1’

   value: {presence: required, castTo: integer} # its value must be cast
   to integer

   }}

   }}

   For another example, see the second example under AggregationRole.

Default
~~~~~~~

The ‘default’ keyword specifies a default field value. Note: the default
value must be within the range or enumeration of acceptable values.

Examples:

-  fieldname: { range: [ 1, unbounded ], default: 5 }

-  fieldname: { value: [ red, white, blue ], default: blue }

HeartbeatAction
~~~~~~~~~~~~~~~

The ‘heartbeatAction’ keyword is provided on the ‘event’ objectName for
heartbeat events only. It provides design time guidance to the service
provider’s heartbeat processing applications (i.e., their watchdog
timers). The syntax and semantics of the ‘heartbeatAction’ keyword are
similar to the ‘action’ keyword except the trigger is specified by the
first field only instead of the first two fields. When the
‘heartbeatAction’ keyword is indicated, the first field is an integer
indicating the number of successively missed heartbeat events. Should
that trigger occur, the remaining fields have the same order, meaning
and optionality as those described for the ‘action’ keyword.

Examples:

-  event: { heartbeatAction: [ 3, vnfDown, RECO-rebootVnf, tcaEventName
      ] }

..

   # whenever the above event occurs, a vnfDown condition is asserted
   and the vnf should be rebooted, plus the indicated tca should be
   generated.

isHomogeneous
~~~~~~~~~~~~~

The isHomogeneous keyword is applied to fields containing
arrayOfNamedHashMap. It lets the receiving event processor know whether
each namedHashMap in the arrayOfNamedHashMap conveys the same data
structure or whether convey different data structures.

isHomogeneous may be set to one of the following (note that lowercase
letters only should be used):

-  true

-  false

Example: The second example for the AggregationRole keyword, shows how
isHomogeneous is used. In that example, the implication is that all
namedHashMapssent through additionalMeasurements would convey
TunnelTraffic data sets and thus the receiving event processor could
convert all the data within additionalMeasurements into a single table.

On the other hand, if isHomogeneous had been set to ‘false’, that would
tell the receiving event processor that different types of data are
being conveyed through additionalMeasurements (e.g., maybe TunnelTraffic
data is sent in some namedHashMaps while QosReport data is sent in other
namedHashMaps) and the receiving event processor would have to process
it accordingly.

key
~~~

The ‘key’ keyword describes a specific key as part of a key-value pair
that may be sent within a keyValuePair keyword (see ‘keyValuePair’
keyword for more explanation and examples).

keyValuePair
~~~~~~~~~~~~

The ‘keyValuePair’ keyword describes a specific key-value pair that may
be sent within a hashMap structure (i.e., a VES field with the hashMap
datatype) or a keyValuePairString structure (see the keyValuePairString
keyword for more information).

Within keyValuePair is a single ‘key’ and a single ‘value’ keyword, each
of which may be decorated with other keywords specified in this document
(e.g., with ‘presence’, ‘range’ and other relevant keywords).

Examples:

-  The following specifies an additionalInformation field within VES,
      which is of type hashMap:

..

   additionalInformation: {presence: optional, structure: {

   keyValuePair: {presence: required, structure: {

   key: {presence: required, value: name1},

   value: {presence: required}

   }},

   keyValuePair: {presence: optional, structure: {

   key: {presence: required, value: name2},

   value: {presence: required}

   }}

   }}

keyValuePairString
~~~~~~~~~~~~~~~~~~

The ‘keyValuePairString’ keyword describes the key-value pairs to be
communicated through a string (e.g., in the VES Syslog Fields
‘syslogSData’ or ‘additionalFields’ strings). This keyword takes three
parameters:

-  the first parameter specifies the character used to delimit (i.e., to
      separate) the key-value pairs. If a space is used as a delimiter,
      it should be communicated within single quotes as ‘ ‘; otherwise,
      the delimiter character should be provided without any quotes.

-  The second parameter specifies the characters used to separate the
      keys and values. If a space is used as a separator, it should be
      communicated within single quotes as ‘ ‘; otherwise, the separator
      character should be provided without any quotes.

-  The third parameter is a “sub-keyword” (i.e., it is used only within
      ‘keyValuePairString’) called ‘keyValuePairs: [ ]’. Within the
      square brackets, a list of ‘keyValuePair’ keywords can be provided
      (see the ‘keyValuePair keyword for more information).

Examples:

-  The following specifies an additionalFields string which is stuffed
      with ‘key=value’ pairs delimited by the pipe (‘|’) symbol as in
      (“key1=value1|key2=value2|key3=value3…”).

additionalFields: {presence: required, keyValuePairString: {|, =,
keyValuePairs: [

keyValuePair: {presence: required, structure: {

key: {presence: required, value: someKeyName},

value: {presence: required, range: [0, 100]}

} },

keyValuePair: {presence: optional, structure: {

key: {presence: required, value: someOtherKeyName},

value: {presence: required, value [red, white, blue]}

} }

] } }

Presence
~~~~~~~~

The ‘presence’ keyword may be defined as ‘required’ or ‘optional’. If
not provided, the element is assumed to be ‘optional’.

Examples

-  element: { presence: required } # element must be present

-  element: { presence: optional } # element is optional

-  element: { value: blue } # by omitting a presence definition, the

..

   element is assumed to be optional

Range
~~~~~

The ‘range’ keyword applies to fields (i.e., simpleTypes); indicates the
value of the field is a number within a specified range of values from
low to high (inclusive of the indicated values). . ‘range:’ is followed
by two parameters in square brackets:

-  the first parameter conveys the minimum value

-  the second parameter conveys the maximum value or ‘unbounded’

The keyword ‘unbounded’ is supported to convey an unbounded upper limit.
Note that the range cannot override any restrictions defined in the VES
Common Event Format.

Examples:

-  fieldname: { range: [ 1, unbounded ] }

-  fieldname: { range: [ 0, 3.14 ] }

Structure
~~~~~~~~~

The ‘structure’ keyword indicates that the element is a complexType
(i.e., an object) and is followed by curly braces containing that
object.

Example:

-  objectName: { structure: {

element1: { },

element2: { },

anotherObject: { structure: {

element3: { },

element4: { }

} }

} }

Units
~~~~~

The ‘units’ qualifier may be applied to values provided in VES Common
Event Format extensible field structures. The ‘units’ qualifier
communicates the units (e.g., megabytes, seconds, Hz) that the value is
expressed in. Note: the ‘units’ should not contain any space characters
and should be expressed in camel casing (e.g., use ‘numberOfPorts’, not
‘number_of_ports’ nor ‘number of ports’).

Example:

-  additionalInformation: { presence: optional, structure: {

keyValuePair: {presence: required, structure: {

key: {presence: required, value: pilotNumberPoolSize},

value: {presence: required, units: megaBytes}

}}

}}

Value
~~~~~

The ‘value’ keyword applies to fields (i.e., simpleTypes); indicates a
single value or an enumeration of possible values. If not provided, it
is assumed the value will be determined at runtime. Note that the
declared value cannot be inconsistent with restrictions defined in the
VES Common Event Format (e.g., it cannot add an enumerated value to an
enumeration defined in the Common Event Format, but it can subset the
defined enumerations in the Common Event Format).

Values that are strings containing spaces should always be indicated in
single quotes.

Examples:

-  fieldname: { value: x } # the value is ‘x’

-  fieldname: { value: [ x, y, z ] } # the value is either ‘x’, ‘y’, or
      ‘z’

-  fieldname: { presence: required } # the value will be provided at
      runtime

-  fieldname: { value: ‘error state’ } # the value is the string within
      the single quotes

Complex Conditions
------------------

Conditions Document
~~~~~~~~~~~~~~~~~~~

After all events have been defined, the YAML file may provide a YAML
document delimited by ‘---‘ and ‘…’, which specifies complex conditions
defined in terms of other conditions, which were asserted in the action
qualifiers of the preceding event definitions. For example:

   ---

   # Event Registration for eventName ‘name1’

   event: {presence: required, action: [any, any, A, null], structure: {

   # details omitted

   }}

   ...

   ---

   # Event Registration for eventName ‘name2’

   event: {presence: required, structure: {

   commonEventHeader: {presence: required, structure: {

   # details omitted

   }}

   measurementFields: {presence: required, structure: {

cpuUsageArray: {presence: required, array: [

cpuUsage: {presence: required, structure: {

cpuIdentifier: {presence: required},

percentUsage: {presence: required, action: [90, up, B, null]}

}}

]},

# details omitted

   }}

   }}

   ...

   ---

   # Complex Conditions

   conditions: [

   conditionC: { defined in terms of A and B, details omitted },

   conditionD: { defined in terms of A, B and C details omitted }

   ]

   ...

Conditions Syntax and Semantics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The YAML ‘conditions’ document begins with the keyword ‘conditions’
followed by a colon and square brackets. Each condition is then defined
within the square brackets. Commas are used to separate conditions.

Each condition is expressed as follows:

conditionName: *logical expression in terms of other conditions*

Logical Expressions
~~~~~~~~~~~~~~~~~~~

The following logical operators are supported:

-  &&: which is a logical AND

-  \||, which is a logical OR

In addition parentheses may be used to group expressions.

Example logical expression:

   (A && B) \|\| (C && D)

Where A, B, C and D are named conditions expressed earlier in the YAML
file.

Because YAML doesn’t support the above operators, the whole expression
needs to be enclosed in double quotes.

Example for a condition named ‘ConditionP’:

---

conditions: [

conditionP: "B \|\| (C && D)"

]

...

Alternatively, Logical ORs and ANDs can be expressed using a syntax
defined by `metacpan.org <https://metacpan.org/pod/YAML::Logic>`__ for
Perl. Using this syntax, the example above looks like this:

---

conditions: [

conditionP: [or, [B, [and, [C, D]]]]

]

...

In the above syntax, both ORs and ANDs are provided as two nested
arrays, where the outer array consists of two items:

-  The desired operator as either ‘or’ or ‘and’

-  The inner array which consists of the conditions to be OR’d or AND’d
   together

Note1: more than two conditions may be or’d or and’d (e.g., “B \|\| C
\|\| D” or “B && C && D”)

Note2: however expressed by YAML, parsers must be developed to make use
of the above expressions.

Time Based Qualifiers
~~~~~~~~~~~~~~~~~~~~~

Time based rules may be established using a timing keyword as follows:

---

conditions: [

PersistentB1: {

timing: {

condition: B,

occurrences: 3,

interval: 300,

units: seconds

}

}

]

...

This means that if condition B occurs 3 (or more) times in 300 seconds
(e.g., 5 minutes), then condition ‘PersistentB1’ has occurred.

Array Item Qualifiers
~~~~~~~~~~~~~~~~~~~~~

This ‘arrayItems’ keyword defines conditions evaluated across the items
in an array. In the example below, the condition is named ‘AnyOfC’:

---

conditions: [

AnyOfC: {

arrayItems: {

condition: C,

arrayItem: measurements.cpuUsageArray.cpuUsage,

scope: anyOf

}

}

]

...

This means that if condition C occurs on any cpuUsage structure within
the cpuUsageArray, then the condition ‘AnyOfC’ is in effect.

Note the dotted notation used to convey the path to the arrayItem with
respect to the encapsulating domain field block.

Scope may have the values ‘anyOf’ or ‘allOf’

Mathematical Expressions
~~~~~~~~~~~~~~~~~~~~~~~~

Mathematical expressions evaluate to a number, using ‘(‘, ‘)’, ‘+’, ‘-‘,
‘*’, ‘/’ and variables of the form $variablePath where the variablePath
is defined with respect to the encapsulating domain field block.

Mathematical expressions are specified as "${mathematicalExpress}".
Because YAML does not support mathematical operators, the expressions
must be enclosed within double quotes.

---

conditions: [

PersistentB2: {

timing: {

condition: B,

occurrences: 3,

interval: "${60 \* $measurements.measurementInterval}",

units: seconds

}

}

]

...

This means that if condition B occurs 3 (or more) times in an interval
defined by 60 times the measurementInterval (e.g., if the
measurementInterval was expressed in seconds as 5, then this would
evaluate to 300), then condition ‘PersistentB2’ is in effect.

Rules
-----

Rules Document
~~~~~~~~~~~~~~

After all events and conditions have been defined, the YAML file may
conclude with a final YAML document delimited by ‘---‘ and ‘…’, which
defines rules based on the named ‘conditions’ asserted previously. For
example:

   ---

   # Event Registration for eventName ‘name1’

   event: {presence: required, action: [any, any, A, null], structure: {

   # details omitted

   }}

   ...

   ---

   # Event Registration for eventName ‘name2’

   event: {presence: required, structure: {

   commonEventHeader: {presence: required, structure: {

   # details omitted

   }}

   measurementFields: {presence: required, structure: {

cpuUsageArray: {presence: required, array: [

cpuUsage: {presence: required, structure: {

cpuIdentifier: {presence: required},

percentUsage: {presence: required, action: [90, up, B, null]}

}}

]},

# details omitted

   }}

   }}

   ...

   ---

   # Complex Conditions

   conditions: [

   C: { details omitted }

   ]

   ...

   ---

   # Rules

   rules: [

   # defined based on conditions ‘A’, ‘B’ and ‘C’ - details omitted

   ]

   ...

Rules Syntax and Semantics
~~~~~~~~~~~~~~~~~~~~~~~~~~

The YAML ‘rules’ document begins with the keyword ‘rules’ followed by a
colon and square brackets. Each rule is then defined within the square
brackets (of ‘rules’). Commas are used to separate rule structure.

Each rule structure is expressed as follows:

rule: {

trigger: *logical expression in terms of conditions*,

microservices: [ *microservice1, microservice2, microservice3…* ]

events: [e*ventName1, eventName2, eventName3…* ],

}

Notes:

-  All referenced eventNames should be defined within the YAML.

-  At least one microservice or event should be specified, and both
   microservices and events may be specified.

-  For information about microservices, see section 3.1.1 bullet number
   4.

Triggers
~~~~~~~~

Triggers may be as simple as a named condition, or they may be logical
expressions in terms of other conditions using the same syntax as used
by the complex conditions described above. For example:

rules: [

ruleName1: {

trigger: A,

eventss: [eventName1],

microservices: [rebootVm]

},

ruleName2: {

trigger: "B \|\| (C && D)",

microservices: [scaleOut]

}

]

Note: when microservices are defined in terms of multiple event
conditions, the designer should take care to consider whether the target
of the microservice is clear (e.g., which VNF or VM instance to perform
the action on). Future versions of this document may provide more
clarity.

YAML Examples
=============

An example YAML file is provided below which registers some events for a
hypothetical VNF. Note: some of the lines have been manually
wrapped/indented to make it easier to read. Please ignore the section
breaks that interrupt this single file; they were added to make it
easier to rapidly find examples of different types of events.

Fault
-----

---

# registration for Fault_vMrf_alarm003

# Constants: the values of domain, eventName, priority, vfstatus

# , version, alarmCondition, eventSeverity, eventSourceType,

# faultFieldsVersion, specificProblem,

# Variables (to be supplied at runtime) include: eventId,
lastEpochMicrosec,

# reportingEntityId, reportingEntityName, sequence, sourceId,
sourceName,

# startEpochMicrosec

event: {presence: required, action: [ any, any, alarm003,
RECO-rebuildVnf ],

structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: fault},

eventName: {presence: required, value: Fault_Vmrf-Nokia_Alarm003},

eventId: {presence: required},

nfNamingCode: {value: mrfx},

priority: {presence: required, value: Medium},

reportingEntityId: {presence: required},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceId: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

faultFields: {presence: required, structure: {

alarmCondition: {presence: required, value: alarm003},

eventSeverity: {presence: required, value: MAJOR},

eventSourceType: {presence: required, value: virtualNetworkFunction},

faultFieldsVersion: {presence: required, value: 3.0},

specificProblem: {presence: required, value: "Configuration file was
corrupt or

not present"},

vfStatus: {presence: required, value: "Requesting Termination"}

}}

}}

...

---

# registration for clearing Fault_vMrf_alarm003Cleared

# Constants: the values of domain, eventName, priority,

# , version, alarmCondition, eventSeverity, eventSourceType,

# faultFieldsVersion, specificProblem,

# Variables (to be supplied at runtime) include: eventId,
lastEpochMicrosec,

# reportingEntityId, reportingEntityName, sequence, sourceId,

# sourceName, startEpochMicrosec, vfStatus

event: {presence: required, action: [ any, any, alarm003, Clear ],
structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: fault},

eventName: {presence: required, value:
Fault_Vmrf-Nokia_Alarm003Cleared},

eventId: {presence: required},

nfNamingCode: {value: mrfx},

priority: {presence: required, value: Medium},

reportingEntityId: {presence: required},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceId: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

faultFields: {presence: required, structure: {

alarmCondition: {presence: required, value: alarm003},

eventSeverity: {presence: required, value: NORMAL},

eventSourceType: {presence: required, value: virtualNetworkFunction},

faultFieldsVersion: {presence: required, value: 3.0},

specificProblem: {presence: required, value: "Valid configuration file
found"},

vfStatus: {presence: required, value: "Requesting Termination"}

}}

}}

...

Heartbeat
---------

---

# registration for Heartbeat_vMRF

# Constants: the values of domain, eventName, priority, version

# Variables (to be supplied at runtime) include: eventId,
lastEpochMicrosec,

# reportingEntityId, reportingEntityName, sequence, sourceId,
sourceName,

# startEpochMicrosec

event: {presence: required, heartbeatAction: [3, vnfDown,
RECO-rebuildVnf],

structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: heartbeat},

eventName: {presence: required, value: Heartbeat_Vmrf-Nokia},

eventId: {presence: required},

nfNamingCode: {value: mrfx},

priority: {presence: required, value: Normal},

reportingEntityId: {presence: required},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceId: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

heartbeatFields: {presence: optional, structure:{

        heartbeatFieldsVersion: {presence: required, value: 2.0},

        heartbeatInterval: {presence: required, range: [ 15, 300 ],
default: 60 }

}}

}}

...

Measurements
------------

To see how additionalMeasurements may be sent in a way that is easy for
service providers to process without custom logic, please see the
following keywords: aggregationRole, castTo and isHomogeneous. In
particular, see the second example under aggergationRole.

---

# registration for Measurement_vMRF

# Constants: the values of domain, eventName, priority, version,

# measurementFieldsVersion,
additionalMeasurements.namedArrayOfFields.name,

# Variables (to be supplied at runtime) include: eventId,
reportingEntityName, sequence,

# sourceName, start/lastEpochMicrosec, measurementInterval,

# concurrentSessions, requestRate, numberOfMediaPortsInUse,

# cpuUsageArray.cpuUsage,cpuUsage.cpuIdentifier, cpuUsage.percentUsage,

# additionalMeasurements.namedArrayOfFields.arrayOfFields,

# vNicPerformance.receivedOctetsAccumulated,

# vNicPerformance.transmittedOctetsAccumulated,

# vNicPerformance.receivedTotalPacketsAccumulated,

# vNicPerformance.transmittedTotalPacketsAccumulated,

# vNicPerformance.vNicIdentifier, vNicPerformance.receivedOctetsDelta,

# vNicPerformance.receivedTotalPacketsDelta,

# vNicPerformance.transmittedOctetsDelta,

# vNicPerformance.transmittedTotalPacketsDelta,

# vNicPerformance.valuesAreSuspect, memoryUsageArray.memoryUsage,

# memoryUsage.memoryConfigured, memoryUsage.vmIdentifier,

# memoryUsage.memoryUsed, memoryUsage.memoryFree

event: {presence: required, structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: measurement},

eventName: {presence: required, value: Measurement_Vmrf-Nokia},

eventId: {presence: required},

nfNamingCode: {value: mrfx},

priority: {presence: required, value: Normal},

reportingEntityId: {presence: required},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceId: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

measurementFields: {presence: required, structure: {

measurementFieldsVersion: {presence: required, value: 3.0},

measurementInterval: {presence: required, range: [ 60, 3600 ], default:
300},

concurrentSessions: {presence: required, range: [ 0, 100000 ]},

requestRate: {presence: required, range: [ 0, 100000 ]},

numberOfMediaPortsInUse: {presence: required, range: [ 0, 100000 ]},

cpuUsageArray: {presence: required, array: [

cpuUsage: {presence: required, structure: {

cpuIdentifier: {presence: required},

percentUsage: {presence: required, range: [ 0, 100 ],

action: [80, up, CpuUsageHigh, RECO-scaleOut],

action: [10, down, CpuUsageLow, RECO-scaleIn]}

}}

]},

memoryUsageArray: {presence: required, array: [

memoryUsage: {presence: required, structure: {

memoryConfigured: {presence: required, value: 33554432},

memoryFree: {presence: required, range: [ 0, 33554432 ],

action: [100, down, FreeMemLow, RECO-scaleOut],

action: [30198989, up, FreeMemHigh, RECO-scaleIn]},

memoryUsed: {presence: required, range: [ 0, 33554432 ]},

vmIdentifier: {presence: required}

}}

]},

additionalMeasurements: {presence: required, array: [

namedHashMap: {presence: required, structure: {

name: {presence: required, value: licenseUsage},

hashMap: {presence: required, structure: {

keyValuePair: {presence: required, structure: {

key: {presence: required, value: G711AudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: G729AudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: G722AudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: AMRAudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: AMRWBAudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: OpusAudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: H263VideoPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: H264NonHCVideoPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: H264HCVideoPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: MPEG4VideoPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: VP8NonHCVideoPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: VP8HCVideoPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: PLC},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: AEC},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: NR},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: NG},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: NLD},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: G711FaxPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: T38FaxPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: RFactor},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: T140TextPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: EVSAudioPort},

value: {presence: required, range: [ 0, 100000 ],

units: numberOfPorts }

}}

}}

}},

namedHashMap: {presence: required, structure: {

name: {presence: required, value: mediaCoreUtilization},

hashMap: {presence: required, structure: {

keyValuePair: {presence: required, structure: {

key: {presence: required, value: actualAvgAudio},

value: {presence: required, range: [ 0, 255 ],

action: [80, up, AudioCoreUsageHigh, RECO-scaleOut],

action: [10, down, AudioCoreUsageLow, RECO-scaleIn]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: modelAvgAudio},

value: {presence: required, range: [ 0, 100 ],

action: [80, up, AudioCoreUsageHigh, RECO-scaleOut],

action: [10, down, AudioCoreUsageLow, RECO-scaleIn]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: actualMaxAudio},

value: {presence: required, range: [ 0, 255 ]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: modelMaxAudio},

value: {presence: required, range: [ 0, 100 ]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: actualAvgVideo},

value: {presence: required, range: [ 0, 255 ],

action: [80, up, VideoCoreUsageHigh, RECO-scaleOut],

action: [10, down, VideoCoreUsageLow, RECO-scaleIn]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: modelAvgVideo},

value: {presence: required, range: [ 0, 100 ],

action: [80, up, VideoCoreUsageHigh, RECO-scaleOut],

action: [10, down, VideoCoreUsageLow, RECO-scaleIn]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: actualMaxVideo},

value: {presence: required, range: [ 0, 255 ]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: modelMaxVideo},

value: {presence: required, range: [ 0, 100 ]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: actualAvgHcVideo},

value: {presence: required, range: [ 0, 255 ],

action: [80, up, HcVideoCoreUsageHigh, RECO-scaleOut],

action: [10, down, HcVideoCoreUsageLow, RECO-scaleIn]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: modelAvgHcVideo},

value: {presence: required, range: [ 0, 100 ],

action: [80, up, HcVideoCoreUsageHigh, RECO-scaleOut],

action: [10, down, HcVideoCoreUsageLow, RECO-scaleIn]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: actualMaxHcVideo},

value: {presence: required, range: [ 0, 255 ]}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: modelMaxHcVideo},

value: {presence: required, range: [ 0, 100 ]}

}}

}}

}}

]},

vNicPerformanceArray: {presence: required, array: [

vNicPerformance: {presence: required, structure: {

receivedOctetsAccumulated: {presence: required,

range: [ 0, 18446744073709551615 ]},

receivedTotalPacketsAccumulated: {presence: required,

range: [ 0, 18446744073709551615 ]},

receivedOctetsDelta: {presence: required},

range: [ 0, 18446744073709551615 ],

receivedTotalPacketsDelta: {presence: required,

range: [ 0, 18446744073709551615 ]},

transmittedOctetsDelta: {presence: required,

range: [ 0, 18446744073709551615 ]},

transmittedOctetsAccumulated: {presence: required,

range: [ 0, 18446744073709551615 ]},

transmittedTotalPacketsAccumulated: {presence: required,

range: [ 0, 18446744073709551615 ]},

transmittedTotalPacketsDelta: {presence: required,

range: [ 0, 18446744073709551615 ]},

valuesAreSuspect: {presence: required, value: [ true, false ]},

vNicIdentifier: {presence: required}

}}

]}

}}

}}

...

Mobile Flow
-----------

---

# registration for mobileFlow

# Constants: the values of domain, eventName, priority, version

#

# Variables (to be supplied at runtime) include: eventId,
reportingEntityName,

# sequence, sourceName, start/lastEpochMicrosec

#

event: {presence: required, structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: mobileFlow},

eventName: {presence: required, value: MobileFlow_Vxyz-Vendor},

eventId: {presence: required},

nfType: {presence: required, value: sbcx},

priority: {presence: required, value: Normal},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

mobileFlowFields: {presence: required, structure: {

mobileFlowFieldsVersion: {presence: required, value: 3.0},

applicationType: {presence: optional},

appProtocolType: {presence: optional},

appProtocolVersion: {presence: optional},

cid: {presence: optional},

connectionType: {presence: optional},

ecgi: {presence: optional},

flowDirection: {presence: required},

gtpPerFlowMetrics: {presence: required, structure: {

avgBitErrorRate: {presence: required},

avgPacketDelayVariation: {presence: required},

avgPacketLatency: {presence: required},

avgReceiveThroughput: {presence: required},

avgTransmitThroughput: {presence: required},

durConnectionFailedStatus: {presence: optional},

durTunnelFailedStatus: {presence: optional},

flowActivatedBy: {presence: optional},

flowActivationEpoch: {presence: required},

flowActivationMicrosec: {presence: required},

flowActivationTime: {presence: optional},

flowDeactivatedBy: {presence: optional},

flowDeactivationEpoch: {presence: required},

flowDeactivationMicrosec: {presence: required},

flowDeactivationTime: {presence: required},

flowStatus: {presence: required},

gtpConnectionStatus: {presence: optional},

gtpTunnelStatus: {presence: optional},

ipTosCountList: {presence: optional},

ipTosList: {presence: optional},

largePacketRtt: {presence: optional},

largePacketThreshold: {presence: optional},

maxPacketDelayVariation: {presence: required},

maxReceiveBitRate: {presence: optional},

maxTransmitBitRate: {presence: optional},

mobileQciCosCountList: {presence: optional},

mobileQciCosList: {presence: optional},

numActivationFailures: {presence: required},

numBitErrors: {presence: required},

numBytesReceived: {presence: required},

numBytesTransmitted: {presence: required},

numDroppedPackets: {presence: required},

numGtpEchoFailures: {presence: optional},

numGtpTunnelErrors: {presence: optional},

numHttpErrors: {presence: optional},

numL7BytesReceived: {presence: required},

numL7BytesTransmitted: {presence: required},

numLostPackets: {presence: required},

numOutOfOrderPackets: {presence: required},

numPacketErrors: {presence: required},

numPacketsReceivedExclRetrans: {presence: required},

numPacketsReceivedInclRetrans: {presence: required},

numPacketsTransmittedInclRetrans: {presence: required},

numRetries: {presence: required},

numTimeouts: {presence: required},

numTunneledL7BytesReceived: {presence: required},

roundTripTime: {presence: required},

tcpFlagCountList: {presence: optional},

tcpFlagList: {presence: optional},

timeToFirstByte: {presence: required}

}},

gtpProtocolType: {presence: optional},

gtpVersion: {presence: optional},

httpHeader: {presence: optional},

imei: {presence: optional},

imsi: {presence: optional},

ipProtocolType: {presence: required},

ipVersion: {presence: required},

lac: {presence: optional},

mcc: {presence: optional},

mnc: {presence: optional},

msisdn: {presence: optional},

otherEndpointIpAddress: {presence: required},

otherEndpointPort: {presence: required},

otherFunctionalRole: {presence: optional},

rac: {presence: optional},

radioAccessTechnology: {presence: optional},

reportingEndpointIpAddr: {presence: required},

reportingEndpointPort: {presence: required},

sac: {presence: optional},

samplingAlgorithm: {presence: optional},

tac: {presence: optional},

tunnelId: {presence: optional},

vlanId: {presence: optional},

additionalInformation: {presence: optional, array: [

keyValuePair: {presence: required, structure: {

key: {presence: required, value: name1},

value: {presence: required}

}},

keyValuePair: {presence: optional, structure: {

key: {presence: required, value: name2},

value: {presence: required}

}}

]}

}}

}}

...

Sip Signaling
-------------

---

# registration for sipSignaling

# Constants: the values of domain, eventName, priority, version

#

# Variables (to be supplied at runtime) include: eventId,
reportingEntityName,

# sequence, sourceName, start/lastEpochMicrosec

#

event: {presence: required, structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: sipSignaling},

eventName: {presence: required, value: SipSignaling_Vxyz-Vendor},

eventId: {presence: required},

nfType: {presence: required, value: sbcx},

priority: {presence: required, value: Normal},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

sipSignalingFields: {presence: required, structure: {

compressedSIP: {presence: optional},

correlator: {presence: required},

localIpAaddress: {presence: required},

localPort: {presence: required},

remoteIpAddress: {presence: required},

remotePort: {presence: required},

sipSignalingFieldsVersion: {presence: required, value: 2.0},

summarySip: {presence: optional},

vnfVendorNameFields: {presence: required, structure: {

vendorName: {presence: required},

vfModuleName: {presence: optional},

vnfName: {presence: optional}

}},

additionalInformation: {presence: optional, array: [

keyValuePair: {presence: required, structure: {

key: {presence: required, value: name1},

value: {presence: required}

}},

keyValuePair: {presence: optional, structure: {

key: {presence: required, value: name2},

value: {presence: required}

}}

]}

}}

}}

...

Syslog
------

---

# registration for Syslog_vMRF

# Constants: the values of domain, eventName, priority,
lastEpochMicrosec, version,

# syslogFields.syslogFieldsVersion, syslogFields.syslogTag

# Variables include: eventId, lastEpochMicrosec, reportingEntityId,
reportingEntityName,

# sequence, sourceId, sourceName, startEpochMicrosec,

# syslogFields.eventSourceHost, syslogFields.eventSourceType,

# syslogFields.syslogFacility, syslogFields.syslogMsg

event: {presence: required, structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: syslog},

eventName: {presence: required, value: Syslog_Vmrf-Nokia},

eventId: {presence: required},

nfNamingCode: {value: mrfx},

priority: {presence: required, value: Normal},

reportingEntityId: {presence: required},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceId: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0},

}},

syslogFields: {presence: required, structure: {

eventSourceHost: {presence: required},

eventSourceType: {presence: required, value: virtualNetworkFunction},

syslogFacility: {presence: required, range: [16, 23]},

syslogSev: {presence: required, value: [Emergency, Alert, Critical,
Error]},

syslogFieldsVersion: {presence: required, value: 3.0},

syslogMsg: {presence: required},

syslogSData: {presence: required, keyValuePairString: {‘ ‘, =,
keyValuePairs: [

keyValuePair: {presence: required, structure: {

key: {presence: required, value: ATTEST},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: DATE_IN},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: DATE_OUT},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: DEST_IN},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: FUNCTION},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: ICID},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: ORIGID},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: ORIG_TN},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: SIP_REASON_HEADER},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: STATE},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: STATUS},

value: {presence: required}

}},

keyValuePair: {presence: required, structure: {

key: {presence: required, value: VERSTAT},

value: {presence: required}

}}

]}} }]

syslogTag: {presence: required, value: vMRF},

additionalFields: {presence: required, keyValuePairString: {|, =,
keyValuePairs: [

keyValuePair: {presence: required, structure: {

key: {presence: required, value: someKeyName},

value: {presence: required}

}},

keyValuePair: {presence: optional, structure: {

key: {presence: required, value: someOtherKeyName},

value: {presence: required}

}}

]}}

}}

}}

...

Voice Quality
-------------

---

# registration for voiceQuality

# Constants: the values of domain, eventName, priority, version

#

# Variables (to be supplied at runtime) include: eventId,
lastEpochMicrosec,

# reportingEntityId, reportingEntityName, sequence, sourceId,

# sourceName, startEpochMicrosec

event: {presence: required, structure: {

commonEventHeader: {presence: required, structure: {

domain: {presence: required, value: voiceQualityFields},

eventName: {presence: required, value: VoiceQuality_Vxyz-Vendor},

eventId: {presence: required},

nfType: {presence: required, value: sbcx},

priority: {presence: required, value: Normal},

reportingEntityName: {presence: required},

sequence: {presence: required},

sourceName: {presence: required},

startEpochMicrosec: {presence: required},

lastEpochMicrosec: {presence: required},

timeZoneOffset: {presence: required},

version: {presence: required, value: 3.0}

}},

voiceQualityFields: {presence: required, structure: {

voiceQualityFieldsVersion: {presence: required, value: 2.0},

calleeSideCodec: {presence: required},

callerSideCodec: {presence: required},

correlator: {presence: required},

remoteIpAddress: {presence: required},

endOfCallVqmSummaries: {presence: required, structure: {

adjacencyName: {presence: required},

endpointDescription: {presence: required},

endpointAverageJitter: {presence: optional},

endpointMaxJitter: {presence: optional},

endpointRtpOctetsLost: {presence: optional},

endpointRtpPacketsLost: {presence: optional},

endpointRtpOctetsDiscarded: {presence: optional},

endpointRtpOctetsReceived: {presence: optional},

endpointRtpOctetsSent: {presence: optional},

endpointRtpPacketsDiscarded: {presence: optional},

endpointRtpPacketsReceived: {presence: optional},

endpointRtpPacketsSent: {presence: optional},

localAverageJitter: {presence: optional},

localMaxJitter: {presence: optional},

localAverageJitterBufferDelay: {presence: optional},

localMaxJitterBufferDelay: {presence: optional},

localRtpOctetsDiscarded: {presence: optional},

localRtpOctetsLost: {presence: optional},

localRtpOctetsReceived: {presence: optional},

localRtpOctetsSent: {presence: optional},

localRtpPacketsDiscarded: {presence: optional},

localRtpPacketsLost: {presence: optional},

localRtpPacketsReceived: {presence: optional},

localRtpPacketsSent: {presence: optional},

mosCqe: {presence: optional},

oneWayDelay: {presence: optional},

packetLossPercent: {presence: optional},

rFactor: {presence: optional},

roundTripDelay: {presence: optional}

}},

phoneNumber: {presence: required},

midCallRtcp: {presence: required},

vendorVnfNameFields: {presence: required, structure: {

vendorName: {presence: required},

vfModuleName: {presence: optional},

vnfName: {presence: optional}

}},

additionalInformation: {presence: optional, array: [

keyValuePair: {presence: required, structure: {

key: {presence: required, value: name1},

value: {presence: required}

}},

keyValuePair: {presence: optional, structure: {

key: {presence: required, value: name2},

value: {presence: required}

}}

]}

}}

}}

...

.. _rules-1:

Rules
-----

---

#Rules

Rules: [

ruleName1: {

trigger: "CpuUsageHigh \|\| FreeMemLow \|\| AudioCoreUsageHigh \|\|

VideoCoreUsageHigh \|\| HcVideoCoreUsageHigh",

microservices: [scaleOut]

},

ruleName2: {

trigger: "CpuUsageLow && FreeMemHigh && AudioCoreUsageLow &&

VideoCoreUsageLow && HcVideoCoreUsageLow",

microservices: [scaleIn]

}

]

...

Appendix: Historical Change Log
===============================

For the latest changes, see the Change Block just before the Table of
Contents.

+-----------------------+-----------------------+-----------------------+
| Date                  | Revision              | Description           |
+-----------------------+-----------------------+-----------------------+
| 3/15/2017             | 1.0                   | This is the initial   |
|                       |                       | release of the VES    |
|                       |                       | Event Registration    |
|                       |                       | document.             |
+-----------------------+-----------------------+-----------------------+
| 3/22/2017             | 1.1                   | -  Changed the        |
|                       |                       |       ‘alert’         |
|                       |                       |       qualifier to    |
|                       |                       |       ‘action’ and    |
|                       |                       |       added support   |
|                       |                       |       for conditions  |
|                       |                       |       that will       |
|                       |                       |       trigger rules.  |
|                       |                       |                       |
|                       |                       | -  Formatted the      |
|                       |                       |       document with   |
|                       |                       |       more sections   |
|                       |                       |       and             |
|                       |                       |       subsections.    |
|                       |                       |                       |
|                       |                       | -  Defined the syntax |
|                       |                       |       and semantics   |
|                       |                       |       for condition   |
|                       |                       |       based rules.    |
|                       |                       |                       |
|                       |                       | -  Fixed the YAML     |
|                       |                       |       examples.       |
+-----------------------+-----------------------+-----------------------+
| 3/27/2017             | 1.2                   | -  Clarified the      |
|                       |                       |       audience of the |
|                       |                       |       document and    |
|                       |                       |       the             |
|                       |                       |       expectations    |
|                       |                       |       for vendors.    |
|                       |                       |                       |
|                       |                       | -  Changed the order  |
|                       |                       |       of fields in    |
|                       |                       |       the action      |
|                       |                       |       keyword.        |
|                       |                       |                       |
|                       |                       | -  Updated the YAML   |
|                       |                       |       examples.       |
|                       |                       |                       |
|                       |                       | -  Wordsmithed        |
|                       |                       |       throughout.     |
+-----------------------+-----------------------+-----------------------+
| 3/31/2017             | 1.3                   | -  Generalized the    |
|                       |                       |       descriptions    |
|                       |                       |       from an ASDC,   |
|                       |                       |       ECOMP and       |
|                       |                       |       AT&T-specific   |
|                       |                       |       interaction     |
|                       |                       |       with a VNF      |
|                       |                       |       vendor, to a    |
|                       |                       |       generic Service |
|                       |                       |       Provider        |
|                       |                       |       interaction     |
|                       |                       |       with a VNF      |
|                       |                       |       vendor.         |
|                       |                       |                       |
|                       |                       | -  Wordsmithed        |
|                       |                       |       throughout.     |
|                       |                       |                       |
|                       |                       | -  Added a ‘default’  |
|                       |                       |       qualifier       |
|                       |                       |                       |
|                       |                       | -  Fixed syntax and   |
|                       |                       |       semantic        |
|                       |                       |       inconsistencies |
|                       |                       |       in the Rules    |
|                       |                       |       section         |
|                       |                       |                       |
|                       |                       | -  Brought all        |
|                       |                       |       examples into   |
|                       |                       |       compliance with |
|                       |                       |       v5.0            |
|                       |                       |                       |
|                       |                       | -  Added a heartbeat  |
|                       |                       |       example         |
|                       |                       |                       |
|                       |                       | -  Modified the       |
|                       |                       |       measurement     |
|                       |                       |       example         |
|                       |                       |                       |
|                       |                       | -  Modified the       |
|                       |                       |       syslog example  |
|                       |                       |                       |
|                       |                       | -  Added two complex  |
|                       |                       |       rules           |
+-----------------------+-----------------------+-----------------------+
| 4/14/2017             | 1.4                   | -  Wordsmithed        |
|                       |                       |       throughout      |
|                       |                       |                       |
|                       |                       | -  Action keyword:    |
|                       |                       |       clarified use   |
|                       |                       |       of ‘up’, ‘down’ |
|                       |                       |       and ‘at’        |
|                       |                       |       triggers;       |
|                       |                       |       clarified the   |
|                       |                       |       specification   |
|                       |                       |       and use of      |
|                       |                       |       microservices   |
|                       |                       |       directives at   |
|                       |                       |       design time and |
|                       |                       |       runtime,        |
|                       |                       |       clarified the   |
|                       |                       |       use of tca’s    |
|                       |                       |                       |
|                       |                       | -  HeartbeatAction    |
|                       |                       |       keyword: Added  |
|                       |                       |       the             |
|                       |                       |       heartbeatAction |
|                       |                       |       keyword         |
|                       |                       |                       |
|                       |                       | -  Value keyword:     |
|                       |                       |       clarified the   |
|                       |                       |       communicaton of |
|                       |                       |       strings         |
|                       |                       |       containing      |
|                       |                       |       spaces.         |
|                       |                       |                       |
|                       |                       | -  Rules: corrected   |
|                       |                       |       the use of      |
|                       |                       |       quotes in       |
|                       |                       |       examples        |
|                       |                       |                       |
|                       |                       | -  Examples: added    |
|                       |                       |       the             |
|                       |                       |       heartbeatAction |
|                       |                       |       keyword on the  |
|                       |                       |       heartbeat event |
|                       |                       |       example; also   |
|                       |                       |       corrected use   |
|                       |                       |       of quotes       |
|                       |                       |       throughout.     |
+-----------------------+-----------------------+-----------------------+
| 10/3/2017             | 1.5                   | -  Back of Cover      |
|                       |                       |    Page: updated the  |
|                       |                       |    license and        |
|                       |                       |    copyright notice   |
|                       |                       |    to comply with     |
|                       |                       |    ONAP guidelines    |
|                       |                       |                       |
|                       |                       | -  Section 3.1: Added |
|                       |                       |       a ‘Units’       |
|                       |                       |       qualifier       |
|                       |                       |                       |
|                       |                       | -  Examples: updated  |
|                       |                       |       the examples to |
|                       |                       |       align with VES  |
|                       |                       |       5.4.1           |
+-----------------------+-----------------------+-----------------------+
| 10/31/2017            | 1.6                   | -  Added              |
|                       |                       |    KeyValuePairString |
|                       |                       |    keyword to handle  |
|                       |                       |    strings which have |
|                       |                       |    delimited          |
|                       |                       |    key-value pairs    |
|                       |                       |    within them.       |
|                       |                       |                       |
|                       |                       | -  Updated the syslog |
|                       |                       |       example to show |
|                       |                       |       the use of      |
|                       |                       |       KeyValuePairStr |
|                       |                       | ing                   |
|                       |                       |                       |
|                       |                       | -  Updated the syslog |
|                       |                       |       example to      |
|                       |                       |       align syslogSev |
|                       |                       |       with VES 5.4.1  |
|                       |                       |                       |
|                       |                       | -  Added examples for |
|                       |                       |       mobile flow,    |
|                       |                       |       sip signaling   |
|                       |                       |       and voice       |
|                       |                       |       quality         |
|                       |                       |                       |
|                       |                       | -  Added sections     |
|                       |                       |       within the      |
|                       |                       |       examples to     |
|                       |                       |       facilitate      |
|                       |                       |       rapid access to |
|                       |                       |       specific types  |
|                       |                       |       of example      |
|                       |                       |       events          |
|                       |                       |                       |
|                       |                       | -  Wordsmithed the    |
|                       |                       |    Introduction       |
+-----------------------+-----------------------+-----------------------+
