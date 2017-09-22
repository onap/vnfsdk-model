.. This work is licensed under a Creative Commons Attribution 4.0 International License.

.. toctree::
   :maxdepth: 1

files/vnfsdk-apis

VNFSDK
======

Documentation Outline
---------------------

Components
----------

VNF Packaging Model/Blueprint 
-----------------------------

VNF product model/blueprint provides a declarative way to define deployment, operational and functional attributes of a VNF product. The VNF product is defined in terms of deployment time requirements and dependencies and exposed telemetry indicator definitions.

The deployment time requirements and dependencies define any and all compute infrastructure needs of the VNF product, such as specific hardware architecture, on-chip features, instruction set availability and hypervisor capabilities.

The telemetry indicator definitions define a set of default indicators exposed by a given VNF product for use by monitoring and assurance tools. This list can be extended and customized once a given VNF product is on-boarded and instantiated at run-time.

The VNF product model is specified using the TOSCA NFV simple profile. It is persisted, along with the product executables and data, using TOSCA CSAR files.

VNF SDK Tools 
-------------

VNF SDK tools provide VNF product DevOps engineers with command line tools and client side API language bindings to define the VNF product model and package content. The following tools are included...

•	VNF Package Builder - creates a CSAR file based on inputs provided by the VNF product DevOps engineer

•	VNF Package Validator - validates the content of the VNF packages to ensure that everything has been built correctly

•	VNF Package Extractor - extracts VNF product model and executables from the CSAR file

•	VNF Package Parser - translates VNF proeduct blueprint into a format consumable by OPEN-O components

•	VNF Package Dry Run - performs a "dry run" install to ensure that the package can be deployed during instantiation

VNF Validation Tests 
--------------------

TBD

User Guides
----------- 

VNF Package Tools User Guide 
----------------------------

VNF Package Designer, provides VNF product DevOps engineers with a graphical tool to define the VNF product model and package content. It is made available as part of the VNF Supplier SDK tools.The package designer makes use of the VNF SDK command line interfaces (CLIs) and client-side API language bindings in order to define the model and the package content. As such, it is functionally equivalent to the VNF SDK tools.

Marketplace User Guide for Operators 
------------------------------------

TBD

Marketplace User Guide for VNF Suppliers 
----------------------------------------

TBD
