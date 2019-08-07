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

Program began with OPNFV Verified Programs

- Supports both self-testing and third-party lab testing

- Initial version will test VIM+NFVI

We propose expanding the program in 2018 to include VNF Compliance

- Requirements and tests defined by ONAP

- Test framework provided by OPNFV(Dovetail) and ONAP VNF Test Platform (VTP)

- Back-end infrastructure provided by Linux Foundation


CVC Structures
==============

|image2|

.. |image2| image:: cvc.png
   :height: 600px
   :width: 600px

.. toctree::
   :maxdepth: 1

   vnf-test-platform.rst
   csar-validation.rst
