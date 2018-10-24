.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017-2018 Huawei Technologies Co., Ltd.

Release Notes
=============

.. note::
   VNF onboarding is a challenge across the industry because of the lack of a
   standard format for VNFs.
   This project provides an ecosystem for ONAP compatible VNFs by:

   * developing tools for vendor CI/CD toolchains
   * developing validation and testing tools

Version: 1.1.0
--------------


:Release Date: 2018-06-07



**New Features**
	* Integration with SDC for VNF Onboarding
	* Functional test support
	* Incorporation of ICE tools for HEAT validation
	* Experimental integration with OPNFV Dovetail
	* Preliminary support for SOL-004
	* Support for HTTPS

**Bug Fixes**
	* Fix localization support

**Known Issues**

**Security Notes**

VNFSDK code has been formally scanned during build time using NexusIQ and all Critical vulnerabilities have been addressed, items that remain open have been assessed for risk and determined to be false positive. The VNFSDK open Critical security vulnerabilities and their risk assessment have been documented as part of the `project <https://wiki.onap.org/pages/viewpage.action?pageId=28377592>`_.

Quick Links:
 	- `VNFSDK project page <https://wiki.onap.org/display/DW/VNF+SDK+Project>`_
 	
 	- `Passing Badge information for VNFSDK <https://bestpractices.coreinfrastructure.org/en/projects/1588>`_
 	
 	- `Project Vulnerability Review Table for VNFSDK <https://wiki.onap.org/pages/viewpage.action?pageId=28377592>`_

**Upgrade Notes**
	* Updated to use Swagger for APIs

**Deprecation Notes**


**Other**


Version: 1.0.0
--------------


:Release Date: 2017-11-16



**New Features**

The VNF SDK project delivers a set of tools designed to expand the VNF
ecosystem for ONAP.

It provides:

* VNF packaging tools, which bundle VNFs into an ONAP-compliant TOSCA CSAR file
* VNF Marketplace, which sits between VNF suppliers and operators. It provides
  a repository for uploading and downloading VNFs and tools to validate package
  consistency.
* VES Collector that may optionally be incorporated into VNFs

VNF SDK works with SDC to facilitate VNF Onboarding.

**Bug Fixes**

N/A

**Known Issues**

`VNFSDK-126 <https://jira.onap.org/browse/VNFSDK-126>`_ : The service 'GET /packageresource/csrs' ignores query parameters

**Security Issues**

N/A

**Upgrade Notes**

N/A

**Deprecation Notes**

N/A

**Other**

N/A
