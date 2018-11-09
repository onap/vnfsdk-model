.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2017-2018 Huawei Technologies Co., Ltd.

VNF SDK Marketplace
-------------------

VNF SDK provides a reference implementation "marketplace" to help vendors
validate and manage VNF packages. It also supports the operator to onboard VNF
into ONAP.The API documentation using swagger can be found at http://hostIP:8702/apidocs
or :download:`json <swagger.json>` :download:`yaml <swagger.yaml>`

|image0|

.. |image0| image:: files/vnfsdk-marketplace.png
   :height: 600px
   :width: 800px

1.  **VNF Repository** is a reference repository for VNFs.

    a. It provides functionalities such as:
        i. Upload/Re-upload VNF
        ii. Download VNF
        iii. Query VNF based on several parameters
        iv. Delete VNF
    b. It also provides an intuitive Graphical user interface to perform above
       activities.

2.  Along with these, VNF SDK also provides hooks to call other tools or
    libraries including **Validation and Function Tests**.

    a. Validation verifies the package structure, mandatory files and their
    format. Currently, the tool performs basic validation to support SDC. In a
    future release, it will also ensure integrity and authenticity of the
    package as described by VNF Requirements.

    b. Function Test provides Robot framework test cases present in each VNF.
    Function test executes those test cases and send the test response back to
    the marketplace. While the framework is in place, actual test development
    is deferred for a future release.

3. **VNF SDK Integration with SDC**

    a. In Amsterdam release, the SDC-UI is being integrated with the VNF
    Repository backend. It provides seamless download, search, view of the VNF
    present in VNF repository. The user can onboard these validated VNF into
    the SDC catalogue.

VNF SDK Marketplace Installation Instructions
---------------------------------------------

1. Download vnfsdk/refrepo from Gerrit
::

  git clone http://gerrit.onap.org/r/vnfsdk/refrepo

2. Goto vnfmarket-be/deployment/install,
::

  Delete old docker images of refrepo (if any).
  Enter command "source .env" to set up few environment variables.
  Enter command "docker-compose up -d"

This will start two Docker containers:

    a. VNF Repository
    b. PostgreSQL database.

Once started, access the Marketplace from your web browser.

3. Connect to http://{host}:8702/onapui/vnfmarket to access the user interface

VNF SDK Tools
-------------

VNF SDK tools provide VNF product DevOps engineers with command line tools and
client side API language bindings to define the VNF product model and package
content. The following tools are included...

•	VNF Package Builder - creates a CSAR file based on inputs provided by the VNF
	product DevOps engineer

•	VNF Package Validator - validates the content of the VNF packages to ensure
	that everything has been built correctly

•	VNF Package Extractor - extracts VNF product model and executables from the
	CSAR file

•	VNF Package Parser - translates VNF product blueprint into a format
	consumable by ONAP components

•	VNF Package Dry Run - performs a "dry run" install to ensure that the package
	can be deployed during instantiation