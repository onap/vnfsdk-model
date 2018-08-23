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

The tools are provided in a form of a shared library (Python module) that can
be used in other projects. A CLI is also provided out-of-the box for DevOps to
use the library with their scripts and automation framework.

Repository Name: vnfsdk/pkgtools

Clone command: git clone https://git.onap.org/vnfsdk/pkgtools

Installation
------------
Python module with CLI is installed by Python pip command. It is possible to
install into a virtual environment (virtualenv).

To install the vnfsdk package tool from source, run the following commands
in the cloned repository directory:

1. pip install -r requirements.txt
    Install all required dependencies
2. pip install .

To install the vnfsdk pkgtools from onap hosted pypi repository, run the
following commands in a python virtual environment:

1. pip install -i https://nexus3.onap.org/repository/PyPi.release/simple --extra-index-url https://pypi.org/simple vnfsdk

Use VNF SDK package tools
-------------------------
Usage

* Create CSAR by specifying a directory
    vnfsdk csar-create -d DESTINATION [--manifest MANIFEST] [--history HISTORY]
    [--tests TESTS] [--licenses LICENSES] source entry

* Extract CSAR content
    vnfsdk csar-open -d DESTINATION source

* Validate CSAR content
    vnfsdk csar-validate source


All commands have -h switch which displays help and description of all
parameters.
