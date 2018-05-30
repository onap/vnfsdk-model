.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017 Huawei Technologies Co., Ltd.

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

ONAP has adopted the `CII Best Practice Badge Program <https://bestpractices.coreinfrastructure.org/en>`_. The goal of the Beijing release is for all ONAP projects to be close to achieving a CII Passing badge.

VNFSDK code has been formally scanned during build time using NexusIQ and all Critical vulnerabilities found in these scans are closed. The 'VNFSDK security vulnerabilities and their workarounds <https://bestpractices.coreinfrastructure.org/en/projects/1588>'_ have been documented as part of the project.

***Quick Links***
 	- `VNFSDK project page <https://wiki.onap.org/display/DW/VNF+SDK+Project>'_
 	
 	- `Badging information for VNFSDK <https://bestpractices.coreinfrastructure.org/en/projects/1588>'_
 	
 	- `Project Vulnerability Review Table for VNFSDK <https://wiki.onap.org/pages/viewpage.action?pageId=28377592>'_
 	
 	- `Badging Requirements <https://github.com/coreinfrastructure/best-practices-badge#core-infrastructure-initiative-best-practices-badge>'_
 	
 	- `Badging Status for all ONAP projects <https://bestpractices.coreinfrastructure.org/en/projects?q=onap>'_

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

VNFSDK-126: The service 'GET /packageresource/csrs' ignores query parameters

**Security Issues**

N/A

**Upgrade Notes**

N/A

**Deprecation Notes**

N/A

**Other**

VNF onboarding with SDC is manual in Amsterdam.
