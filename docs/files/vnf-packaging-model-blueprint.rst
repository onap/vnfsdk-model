.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017 Huawei Technologies Co., Ltd.

VNF Packaging Model/Blueprint
=============================

VNF product model/blueprint provides a declarative way to define deployment,
operational and functional attributes of a VNF product. The VNF product is
defined in terms of deployment time requirements and dependencies and exposed
telemetry indicator definitions.

The deployment time requirements and dependencies define any and all compute
infrastructure needs of the VNF product, such as specific hardware
architecture, on-chip features, instruction set availability and hypervisor
capabilities.

The telemetry indicator definitions define a set of default indicators exposed
by a given VNF product for use by monitoring and assurance tools. This list
can be extended and customized once a given VNF product is on-boarded and
instantiated at run-time.

The VNF product model is specified using the TOSCA NFV simple profile. It is
persisted, along with the product executables and data, using TOSCA CSAR files.

.. toctree::
   :maxdepth: 1
   :titlesonly:
   :glob:

   pkgtool
   