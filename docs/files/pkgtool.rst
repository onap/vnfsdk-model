.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017-2018 Huawei Technologies Co., Ltd.

VNF Package Tool
================

Provided tools
--------------

* VNF Package Builder - creates a CSAR file based on inputs provided by the VNF
  product DevOps engineer
* VNF Package Validator - validates the content of the VNF packages to ensure
  that everything has been built correctly
* VNF Package Extractor - extracts VNF product model and executables from the
  CSAR file
* VNF Package Parser - translates VNF product blueprint into a format
  consumable by ONAP components

The tools are provided in the form of a shared library (Python module) that can
be used in other projects. A CLI is also provided out-of-the-box for DevOps to
use the library with their scripts and automation framework.

Repository Name: vnfsdk/pkgtools

Clone command: git clone https://git.onap.org/vnfsdk/pkgtools

Installation
------------
Python module with CLI is installed by Python pip command. It is possible to
install into a virtual environment (virtualenv).

To install the vnfsdk package tool from source, run the following commands
in the cloned repository directory:

.. code-block:: bash

 $ pip install -r requirements.txt
 $ pip install .

To install the vnfsdk pkgtools from onap hosted pypi repository, run the
following commands in a python virtual environment:

.. code-block:: bash

  $ pip install -i https://nexus3.onap.org/repository/PyPi.public/simple vnfsdk

Use VNF SDK package tools
-------------------------
Usage

* Create CSAR by specifying a directory

.. code-block:: bash

  vnfsdk [-v] csar-create [-h] -d DESTINATION [--manifest MANIFEST] [--history HISTORY]
  [--tests TESTS] [--licenses LICENSES] [--digest {SHA256,SHA512}]
  [--certificate CERTIFICATE] [--privkey PRIVKEY] source entry

e.g.

.. code-block:: bash

  $ vnfsdk csar-create -d /tmp/helloworld.csar --manifest helloworld.mf --history ChangeLog.txt
  --tests Tests --licenses Licenses --certificate test.crt --privkey test.key --digest SHA256
  ./hello-world/ helloworld.yaml

* Extract CSAR content

.. code-block:: bash

  vnfsdk -v csar-open [-h] -d DESTINATION [--no-verify-cert] source

e.g.

.. code-block:: bash

  $ vnfsdk csar-open -d /tmp/helloworld --no-verify-cert /tmp/helloworld.csar

* Validate CSAR content

.. code-block:: bash

  vnfsdk -v csar-validate [-h] source

e.g.

.. code-block:: bash

  $ vnfsdk csar-validate /tmp/helloworld.csar

All commands have -h switch which displays help and description of all parameters.
