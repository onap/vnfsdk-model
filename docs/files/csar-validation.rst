

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
   :header: "Product", "Release", "Requirement No.", "Description"
   :widths: 5, 5, 5, 50
   :url: https://nexus.onap.org/content/sites/raw/org.onap.vnfsdk.validation/master/VnfActiveRulesTable.csv

Active PNF requirements
=======================

.. csv-table:: Table of PNF active requirements.
   :delim: ;
   :header: "Product", "Release", "Requirement No.", "Description"
   :widths: 5, 5, 5, 50
   :url: https://nexus.onap.org/content/sites/raw/org.onap.vnfsdk.validation/master/PnfActiveRulesTable.csv

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
