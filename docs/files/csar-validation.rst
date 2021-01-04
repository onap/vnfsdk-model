

.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2019 Huawei Technologies Co., Ltd.

.. _csar-validation:

CSAR Compliance check for SOL004 and SOL001
===========================================

ONAP enabled the required compliance check by following VNFREQS and reports the non-compliant entries as errors.

When :ref:`vnf-test-platform` is deployed, by default this testing is enabled with test case name as below

scenario: onap-vtp

testsuite: validation

testcase: csar-validate

And every VNFREQS is modeled as separate test case with name csar-validate-rxxxxx, where xxxxx represents the VNFREQS.

Active VNF requirements
=======================

.. csv-table:: Table of VNF active requirements.
   :delim: ;
   :header: "Release", "Requirement No.", "Description"
   :widths: 5, 5, 50
   :url: https://nexus.onap.org/content/sites/raw/org.onap.vnfsdk.validation/master/VnfActiveRulesTable.csv

Active PNF requirements
=======================

.. csv-table:: Table of PNF active requirements.
   :delim: ;
   :header: "Release", "Requirement No.", "Description"
   :widths: 5, 5, 50
   :url: https://nexus.onap.org/content/sites/raw/org.onap.vnfsdk.validation/master/PnfActiveRulesTable.csv

Casablanca Implemented requirements
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

   * - R-66070
     - The VNF Package MUST include VNF Identification Data to uniquely identify the resource for a given VNF provider. The identification data must include: an identifier for the VNF, the name of the VNF as was given by the VNF provider, VNF description, VNF provider, and version.

   * - R-77707
     - The VNF provider MUST include a Manifest File that contains a list of all the components in the VNF package.

   * - R-77786
     - The VNF Package MUST include all relevant cookbooks to be loaded on the ONAP Chef Server.


Dublin Implemented requirements
===================================

.. list-table::
   :header-rows: 1


   * - **Requirement No.**
     - **Requirement Description**

   * - R-01123
     - The VNF or PNF CSAR package Manifest file MUST contain: VNF or PNF package meta-data,
       a list of all artifacts (both internal and external) entry’s including their respected URI’s,
       an algorithm to calculate a digest and a digest result calculated on the content of each artifacts,
       as specified in ETSI GS NFV-SOL004. The VNF or PNF Package MUST include VNF or PNF Identification Data to uniquely
       identify the resource for a given provider. The identification data must include:
       an identifier for the VNF or PNF, the name of the VNF or PNF as was given by the provider,
       description, provider and version.

   * - R-07879
     - The VNF Package MUST include all relevant playbooks to ONAP to be loaded on the Ansible Server

   * - R-09467
     - The VNF MUST utilize only NCSP standard compute flavors. [5] - compute, virtual storage.

   * - R-10087
     - The VNF/PNF package MUST contain all standard artifacts as specified in ETSI GS NFV-SOL004 including Manifest file, VNFD/PNFD (or Main TOSCA/YAML based Service Template) and other optional artifacts. CSAR Manifest file as per SOL004 - for example ROOT\ MainServiceTemplate.mf

   * - R-13390
     - The VNF provider MUST provide cookbooks to be loaded on the appropriate Chef Server.

   * - R-15837
     - Major TOSCA Types specified in ETSI NFV-SOL001 standard draft.

   * - R-17852
     - The VNFD/PNFD MAY include TOSCA/YAML definitions that are not part of NFV Profile. If provided, these definitions MUST comply with TOSCA Simple Profile in YAML v.1.2.

   * - R-21322
     - The VNF provider MUST provide their testing scripts to support testing as specified in ETSI NFV-SOL004 -
       Testing directory in CSAR.

   * - R-23823
     - The VNF Package MUST include appropriate credentials so that ONAP can interact with the Chef Server

   * - R-26881
     - The VNF provider MUST provide the binaries and images needed to instantiate the VNF (VNF and VNFC images).

   * - R-26885
     - The VNF provider MUST provide the binaries and images needed to instantiate the VNF (VNF and VNFC images) either as:
       Local artifact in CSAR: ROOT\Artifacts\ VNF_Image.bin
       externally referred (by URI) artifact in Manifest file (also may be referred by VNF Descriptor)
       Note: Currently, ONAP doesn’t have the capability of Image management, we upload the image into VIM/VNFM manually.

   * - R-27310
     - The VNF Package MUST include all relevant Chef artifacts (roles/cookbooks/recipes)
       required to execute VNF actions requested by ONAP for loading on appropriate Chef Server.

   * - R-32155
     - The VNFD provided by VNF vendor may use the below described TOSCA interface types.
       An on-boarding entity (ONAP SDC) MUST support them..

   * - R-35854
     - The VNF/PNF Descriptor (VNFD/PNFD) provided by VNF/PNF vendor MUST comply with TOSCA/YAML based Service template for VNF/PNF descriptor specified in ETSI NFV-SOL001.

   * - R-40293
     - The VNF MUST make available (or load on VNF Ansible Server) playbooks that conform to the ONAP requirement.

   * - R-40820
     - The VNF provider MUST enumerate all of the open source licenses their VNF(s) incorporate. CSAR License
       directory as per ETSI SOL004. for example ROOT\Licenses\ License_term.txt

   * - R-43958
     - The VNF Package MUST include documentation describing the tests that were conducted by the VNF provider and the test results.

   * - R-46527
     - A VNFD is a deployment template which describes a VNF in terms of deployment and operational
       behavior requirements. It contains virtualized resources (nodes) requirements as well as connectivity
       and interfaces requirements and MUST comply with info elements specified in ETSI GS NFV-IFA 011.

   * - R-51347
     - The VNF package MUST be arranged as a CSAR archive as specified in TOSCA Simple Profile in YAML 1.2.

   * - R-54356
     - Data types used by NFV node and is based on TOSCA/YAML constructs specified in draft GS NFV-SOL 001.
       The node data definitions/attributes used in VNFD MUST comply.

   * - R-57019
     - The PNF TOSCA CSAR package Manifest file MUST start with the PNF package metadata in the form of name-value pairs. Each pair shall appear on a different line. The name is specified as following: pnfd_provider, pnfd_name, pnfd_release_date_time, pnfd_archive_version

   * - R-65486
     - The VNFD MUST comply with ETSI GS NFV-SOL001 document endorsing the above mentioned NFV Profile and
       maintaining the gaps with the requirements specified in ETSI GS NFV-IFA011 standard.

   * - R-67895
     - The VNFD provided by VNF vendor may use the below described TOSCA capabilities.
       An on-boarding entity (ONAP SDC) MUST support them.

   * - R-87234
     - The VNF/PNF package provided by a VNF/PNF vendor MAY be either with TOSCA-Metadata directory (CSAR Option 1) or without TOSCA-Metadata directory (CSAR Option 2) as specified in ETSI GS NFV-SOL004. On-boarding entity (ONAP SDC) must support both options.

   * - R-95321
     - The VNFD provided by VNF vendor may use the below described TOSCA relationships.
       An on-boarding entity (ONAP SDC) MUST support them.

   * - R-146092
     - The VNF/PNF package Manifest file MUST contain: non-mano artifact set with following ONAP public tag: onap_ves_events, onap_pm_dictionary, onap_yang_module, onap_others

   * - R-293901
     - For a VNF/PNF package, CSAR MUST contains a TOSCA-Metadata directory with the TOSCA.meta metadata file. The TOSCA.meta metadata file MUST include block_0 with the Entry-Definitions keyword pointing to a TOSCA definitions YAML file. Additional keyname extension must be included as following: ETSI-Entry-Manifest, ETSI-Entry-Change-Log

   * - R-787965
     - If the VNF or PNF CSAR Package utilizes Option 2 for package security, then the complete CSAR file MUST be digitally signed with the VNF or PNF provider private key. The VNF or PNF provider delivers one zip file consisting of the CSAR file, a signature file and a certificate file that includes the VNF or PNF provider public key. The certificate may also be included in the signature container if the signature format allows that. The VNF or PNF provider creates a zip file consisting of the CSAR file with .csar extension, signature and certificate files. The signature and certificate files must be siblings of the CSAR file with extensions .cms and .cert respectively.

Frankfurt Implemented requirements
===================================

.. list-table::
   :header-rows: 1


   * - **Requirement No.**
     - **Requirement Description**

   * - R-972082
     - The PNF software information file is included in the package and it MUST be compliant to:
       The file extension which contains the PNF software version must be .yaml
       The PNF software version information must be specified as following: onap_pnf_sw_information: pnf_software_version:  "<version>"

Guilin Implemented requirements
===================================

.. list-table::
   :header-rows: 1


   * - **Requirement No.**
     - **Requirement Description**

   * - R-130206
     - The VNF/PNF package shall contain a Digest (a.k.a. hash) for each of the components of the VNF package. The table of hashes is included in the manifest file, which is signed with the VNF provider private key. In addition, the VNF provider shall include a signing certificate that includes the VNF provider public key, following a pre-defined naming convention and located either at the root of the archive or in a predefined location (e.g. directory).

   * - R-816745
     - The VNF or PNF PROVIDER MUST provide the Service Provider with PM Meta Data (PM Dictionary)
       to support the analysis of PM events delivered to DCAE. The PM Dictionary is to be provided as a separate YAML artifact at onboarding and must follow
       the VES Event Listener Specification and VES Event Registration Specification
       which contain the format and content required.

OCLIP additional parameters in Dublin
=====================================

To run validation of PNF csar additional --pnf parameter must be used.

  oclip --product onap-vtp csar-validate --pnf --csar <path to pnf.csar or package.zip>

Package zip structure
=====================
|image3|

.. |image3| image:: zip_package.png
   :height: 250px
   :width: 260px

Generate certificates
---------------------
  openssl req -nodes -x509 -sha256 -newkey rsa:4096 -keyout "pnf.key" -out "pnf.cert" -days 365 -subj "/C=NL/ST=Zuid Holland/L=Rotterdam/O=Sparkling Network/OU=IT Dept/CN=$(whoami)s Sign Key"

Sign csar file with the private key
-------------------------------
  openssl dgst -sha256 -sign "pnf.key" -out pnf.sha256.cms pnf.csar

Verify signature
----------------
  openssl dgst -sha256 -verify  <(openssl x509 -in "pnf.cert"  -pubkey -noout) -signature pnf.sha256.cms pnf.csar
