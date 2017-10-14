VNF Package Tool
================

Provided tools
--------------

* VNF Package Builder - creates a CSAR file based on inputs provided by the VNF product DevOps engineer
* VNF Package Validator - validates the content of the VNF packages to ensure that everything has been built correctly
* VNF Package Extractor - extracts VNF product model and executables from the CSAR file
* VNF Package Parser - translates VNF product blueprint into a format consumable by ONAP components
* VNF Package Dry Run - performs a "dry run" install to ensure that the package can be deployed during instantiation
 
The tools are provided in a form of a shared library (Python module) that can be used in other projects. A CLI is also provided out-of-the box for DevOps to use the library with their scripts and autoamtion framework.

Repository Name: vnfsdk/pkgtools

Clone command: git clone http://gerrit.onap.org/r/vnfsdk/pkgtools

Installation
------------
Python module with CLI is installed by Python pip command. It is possible to install into a virtual environment (virtualenv).
The following commands are executed in the cloned repository directory:

1. pip install -r requirements.txt
    Install all required dependencies
2. pip install .

Or run the following commands in the cloned repository directory to install:

1. python setup.py install

Install VNF SDK tools package
-----------------------------
Usage

* Create CSAR by specifying a directory
    vnfsdk csar-create -d DESTINATION [--manifest MANIFEST] [--history HISTORY] [--tests TESTS] [--licenses LICENSES] source entry

* Extract CSAR content
    vnfsdk csar-open -d DESTINATION source

* Validate CSAR content
    vnfsdk csar-validate source
 
All commands have -h switch which displays help and description of all paramaters.
