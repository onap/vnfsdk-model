.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017-2018 Huawei Technologies Co., Ltd.
.. _release_notes:

Release Notes
=============

.. note::
   VNF onboarding is a challenge across the industry because of the lack of a
   standard format for VNFs.
   This project provides an ecosystem for ONAP compatible VNFs by:

   * developing tools for vendor CI/CD toolchains
   * developing validation and testing tools

<<<<<<< HEAD   (8d7665 Update release notes version)
=======

 Version: 1.6.2
 --------------

 :Release Date: 2020-11-03
 :Docker Version: 1.6.2

 **New Features**

  * Added active scenario and profile management support
  * Added integration with Robot CSIT tests
  * Enabled auto discovery of test cases from 3rd party tool integration
  * Added support for cnf-conformance test support

 **Bug Fixes**

  * Fix oclip issue and set OCLIP version used during docker image build, to 6.0.0
    - https://jira.onap.org/browse/CLI-325
  * Fix JSON parsing error returned from GET request
    - https://jira.onap.org/browse/VNFSDK-697
  * Fixed rule R130206 handling of CSARs with no TOSCA meta and no Certificate in root directory
    - https://jira.onap.org/browse/VNFSDK-481
  * Fixed rule R816745 that was not reporting error when CMS and TOSCA meta file were present,
     however TOSCA did not contain ETSI-Entry-Certificate
    - https://jira.onap.org/browse/VNFSDK-660

 **Known Issues**

 N/A

 **Security Notes**

 *Fixed Security Issues*

 N/A

 *Known Security Issues*

 N/A

 *Known Vulnerabilities in Used Modules*

 N/A

 **Upgrade Notes**

 N/A

 **Deprecation Notes**

 N/A

 **Other**

 N/A


 Version: 1.6.0
 --------------

 :Release Date: 2020-10-06
 :Docker Version: 1.6.0


 **New Features**
  * R-972082: The PNF software information file is included in the package and it MUST be compliant to:
        The file extension which contains the PNF software version must be .yaml
        The PNF software version information must be specified as following: onap_pnf_sw_information: pnf_software_version:  "<version>"
  * R-130206: The VNF/PNF package shall contain a Digest (a.k.a. hash) for each of the components of the VNF package.
        The table of hashes is included in the manifest file, which is signed with the VNF provider private key.
        In addition, the VNF provider shall include a signing certificate that includes the VNF provider public key,
        following a pre-defined naming convention and located either at the root of the archive or in a predefined location (e.g. directory).
  * R-816745: The VNF or PNF PROVIDER MUST provide the Service Provider with PM Meta Data (PM Dictionary)
        to support the analysis of PM events delivered to DCAE. The PM Dictionary is to be provided as a separate YAML artifact at onboarding and must follow
        the VES Event Listener Specification and VES Event Registration Specification
        which contain the format and content required.
  * Add new field called "warnings" to oclip json response. All ignored errors are now reported as warnings.


 **Bug Fixes**

  * Fixed package integrity issue with non mano arifacts.
     - https://jira.onap.org/browse/VNFSDK-581
  * Fixed VNF/PNF package integrity issue with CMS signature not containing certificate.
     - https://jira.onap.org/browse/VNFSDK-582
  * Fixed bug that was showing errors during validation of CSAR,
    when any other non_mano_artifact_set than onap_pnf_sw_information was present in manifest file.
     - https://jira.onap.org/browse/VNFSDK-585
  * Fixed bug that was generating invalid report when user run validation with all rules and single validation fails.
     - https://jira.onap.org/browse/VNFSDK-586
  * Fixed bug that was causing problem with loading rules properties.
     - https://jira.onap.org/browse/VNFSDK-587
  * Fixed package security SOL004 Option 1 make rule less restrictive as this rule is not implemented in SDC Onboarding
     - https://jira.onap.org/browse/VNFSDK-595
  * Fixed VNFSDK doesn't check if all files in package are listed in manifest file
     - https://jira.onap.org/browse/VNFSDK-583
  * Fixed rule R01123 that was reporting all files in ZIP as not present in manifest
     - https://jira.onap.org/browse/VNFSDK-583
  * Fixed rule R816745 that wasn't sending all exceptions connected with YAML parsing as validation error
     - https://jira.onap.org/browse/VNFSDK-644
  * Fixed rule R816745 that was searching for the path to PM_Dictionary in manifest file under name source,
      instead of Source (starting with a capital letter).
      Now  both versions (source and Source) are accepted by this rule.
     - https://jira.onap.org/browse/VNFSDK-645
  * Fixed rule R130206 CMS and certificate searching and validation mechanism
     - https://jira.onap.org/browse/VNFSDK-595


 **Known Issues**

 N/A

 **Security Notes**

 * Fixed Security Issues*

  * Upgraded from java 8 to java 11
    - https://jira.onap.org/browse/VNFSDK-646
  * Added non-vulnerable log4j version
    - https://jira.onap.org/browse/VNFSDK-553
  * Update certs expiration date from default 30 days to 730 days (2 years)
    - https://jira.onap.org/browse/VNFSDK-650


 *Known Security Issues*

 N/A

 *Known Vulnerabilities in Used Modules*

 **Upgrade Notes**

 N/A

 **Deprecation Notes**

 N/A

 **Other**

 N/A


>>>>>>> CHANGE (f2700b Release notes doc job triggers)
 Version: 1.5.2
 --------------

 :Release Date: 2020-05-26
 :Docker Version: 1.5.2


 **New Features**
  * Pods are enabled to run as non root user
  * Direct Vulnerability issues are addressed
  * HTTPS enabled in OOM deployment
  * Added VTP2OVP result translation tool to support OVP 2019.12
  * VTP architecture is contributed into LFN CNTT under VNF testing framework
  * VTP REST API is contributed to TMF v19.5 specifications 704, 706, 707, 708, 709, 710


 **Bug Fixes**

 N/A

 **Known Issues**

 N/A

 **Security Notes**

 *Fixed Security Issues*

 *Known Security Issues*

 N/A

 *Known Vulnerabilities in Used Modules*

 **Upgrade Notes**

 N/A

 **Deprecation Notes**

 N/A

 **Other**

 N/A


Version: 1.4.0
--------------


:Release Date: 2019-10-07
:Docker Version: 1.4.0



**New Features**
    * TOSCA based VNF validation enabled for supporting OVP & CVC
    * TOSCA based VNF compliance check based on some operators requirements
    * SDC now integrated VNFSDK VTP on VNF validation
    * ETSI SOL004 security check (CMS signature validation) enabled
    * Code quality improvement(e.g. replace the Jackson to Gson, sonar issue fix)
    * A C++ implement of VES spec 7.0.1 on ves-agent.

**Bug Fixes**

N/A

**Known Issues**

N/A

**Security Notes**

*Fixed Security Issues*

*Known Security Issues*

    * In default deployment VNFSDK (refrepo) exposes HTTP port 30297 outside of cluster. [`OJSI-154 <https://jira.onap.org/browse/OJSI-154>`_]
    * CVE-2019-12126 - demo-vnfsdk-vnfsdk exposes JDWP port 8000 on localhost which allows to gain root privileges inside the container [`OJSI-88 <https://jira.onap.org/browse/OJSI-88>`_]

*Known Vulnerabilities in Used Modules*

**Upgrade Notes**

N/A

**Deprecation Notes**

N/A

**Other**

N/A


Version: 1.3.0
--------------


:Release Date: 2019-05-31



**New Features**
    * VTP (VNF Test Platform) is enabled with scenario and test case execution management
    * ONAP SDC is integrated with VTP for providing the validation as part of VSP on-boarding
    * CSAR validation is enabled with PNF and VNF compliance check for SOL004, SOL001 and VNFREQS
    *

**Bug Fixes**

N/A

**Known Issues**

N/A

**Security Notes**

*Fixed Security Issues*

*Known Security Issues*

    * In default deployment VNFSDK (refrepo) exposes HTTP port 30297 outside of cluster. [`OJSI-154 <https://jira.onap.org/browse/OJSI-154>`_]
    * CVE-2019-12126 - demo-vnfsdk-vnfsdk exposes JDWP port 8000 on localhost which allows to gain root privileges inside the container [`OJSI-88 <https://jira.onap.org/browse/OJSI-88>`_]

*Known Vulnerabilities in Used Modules*

**Upgrade Notes**

N/A

**Deprecation Notes**

N/A

**Other**

N/A

Version: 1.2.0
--------------


:Release Date: 2018-11-30



**New Features**
    * LFN CVC test support
    * Introduce VTP (VNF Test Platform) framework for test
    * Better integration with OPNFV Dovetail (VTP)
    * Experimental integration with OPNFV Dovetail
    * Preliminary implementation of VNF requirements
    * Support CSAR packaging SOL-004 option 1 (CSAR with TOSCA-Metadata directory)
    * Support HPA schema validation

**Bug Fixes**

N/A

**Known Issues**

N/A

**Security Notes**

VNFSDK code has been formally scanned during build time using NexusIQ and all Critical vulnerabilities have been addressed, items that remain open have been assessed for risk and determined to be false positive. The VNFSDK open Critical security vulnerabilities and their risk assessment have been documented as part of the `project <https://wiki.onap.org/pages/viewpage.action?pageId=45298880>`_.

Quick Links:
     - `VNFSDK project page <https://wiki.onap.org/display/DW/VNF+SDK+Project>`_

     - `Passing Badge information for VNFSDK <https://bestpractices.coreinfrastructure.org/en/projects/1588>`_

     - `Project Vulnerability Review Table for VNFSDK <https://wiki.onap.org/pages/viewpage.action?pageId=45298880>`_

**Upgrade Notes**

N/A

**Deprecation Notes**

N/A

**Other**

N/A

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

N/A

**Security Notes**

VNFSDK code has been formally scanned during build time using NexusIQ and all Critical vulnerabilities have been addressed, items that remain open have been assessed for risk and determined to be false positive. The VNFSDK open Critical security vulnerabilities and their risk assessment have been documented as part of the `project <https://wiki.onap.org/pages/viewpage.action?pageId=28377592>`_.

Quick Links:
     - `VNFSDK project page <https://wiki.onap.org/display/DW/VNF+SDK+Project>`_

     - `Passing Badge information for VNFSDK <https://bestpractices.coreinfrastructure.org/en/projects/1588>`_

     - `Project Vulnerability Review Table for VNFSDK <https://wiki.onap.org/pages/viewpage.action?pageId=28377592>`_

**Upgrade Notes**
    * Updated to use Swagger for APIs

**Deprecation Notes**

N/A

**Other**

N/A

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
