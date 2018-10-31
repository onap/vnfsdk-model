.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2018 Huawei Technologies Co., Ltd.

=======================================
VNF SDK Compliance Verification Program
=======================================

.. toctree::
   :maxdepth: 1

Background
==========

LF Networking is providing a testing program to demonstrate SDN/NFV 
capabilities and interoperability.
Program began with OPNFV Verified Program
s
- Supports both self-testing and third-party lab testing

- Initial version will test VIM+NFVI

We propose expanding the program in 2018 to include VNF Compliance

- Requirements and tests defined by ONAP

- Test framework provided by OPNFV(Dovetail)

- Back-end infrastructure provided by Linux Foundation

References
==========

Dovetail Introduction
---------------------

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Dovetail

CVC Structures
==============

|image0|

.. |image0| image:: cvc.png
   :height: 600px
   :width: 600px


Casablanca Implemented requriements
===================================

.. list-table::
   :header-rows: 1


   * - **Requirement No.**
     - **Requirement Description**

   * - R-02454
     - The VNF MUST support the existence of multiple major/minor versions of the VNF software and/or sub-components and interfaces that support both forward and backward compatibility to be transparent to the Service Provider usage.

   * - R-04298
     - The VNF provider MUST provide their testing scripts to support testing.

   * - R-26681
     - The VNF provider MUST provide the binaries and images needed to instantiate the VNF (VNF and VNFC images).

   * - R-35851
     - The VNF Package MUST include VNF topology that describes basic network and application connectivity internal and external to the VNF including Link type, KPIs, Bandwidth, latency, jitter, QoS (if applicable) for each interface.

   * - R-43958
     - The VNF Package MUST include documentation describing the tests that were conducted by the VNF provider and the test results.

   * - R-66070
     - The VNF Package MUST include VNF Identification Data to uniquely identify the resource for a given VNF provider. The identification data must include: an identifier for the VNF, the name of the VNF as was given by the VNF provider, VNF description, VNF provider, and version.

   * - R-77707
     - The VNF provider MUST include a Manifest File that contains a list of all the components in the VNF package.
	 
   * - R-77786
     - The VNF Package MUST include all relevant cookbooks to be loaded on the ONAP Chef Server.
