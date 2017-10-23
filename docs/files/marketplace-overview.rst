VNF SDK Marketplace
==================

VNF SDK provides a reference implementation "marketplace" to help vendors validate and manage VNF packages. It also supports the operator to onboard VNF into ONAP.

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
    b. It also provides an intuitive Graphical user interface to perform above activities.


2. Along with these, VNF SDK also provides hooks to call other tools or libraries including **Validation and Function Tests**.

    a. Validation verifies the package structure, mandatory files and their format. Currently, the tool performs basic validation to support SDC. In a future release, it will also ensure integrity and authenticity of the package as described by VNF Requirements.
    b. Function Test provides Robot framework test cases present in each VNF. Function test executes those test cases and send the test response back to the marketplace. While the framework is in place, actual test development is deferred for a future release.

3. **VNF SDK Integration with SDC**

    a. In Amsterdam release, the SDC-UI is being integrated with the VNF Repository backend. It provides seamless download, search, view of the VNF present in VNF repository. The user can onboard these validated VNF into the SDC catalogue.

