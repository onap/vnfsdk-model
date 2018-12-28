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

VNF Test Platform (VTP)
=======================
Deploy VNF test cases once and trigger it safely from anywhere

Objectives
----------
* LFN/ONAP wants test platform where VNF packages could be certified using ONAP requirements to drive industry adoption
* Provide an platform where vendor/operator can develop, deploy, run test cases and query the results
* Test cases, test results and VNF should be manageable .i,e with authorization, so only user with given roles is allowed to perform operation like VNF package upload/download, run compliance verification tests, allow only specific VIM for specific users, etc.
* Test results should be persisted and should be available for human analysis later via LFN infrastructure.
* Provides test flow where author make flow across different test cases for a given program like compliance verification and  VNFREQS/SOL0004.
* Provide integration with OPNFV dovetail to run test cases across dovetail and VNFSDK.
* Deployable as docker container.

|image0|

.. |image0| image:: VTP.png
   :height: 600px
   :width: 900px

Architecture
------------

|image1|

.. |image1| image:: VTParch.png
   :height: 600px
   :width: 900px

VTP deployment
----------------

Install VTP Backend
~~~~~~~~~~~~~~~~~~~~~~~~~~

export NEXUS_DOCKER_REPO=nexus3.onap.org:10001

export REFREPO_TAG=1.2.1-STAGING-20181228T020411Z

export POSTGRES_TAG=latest

export MTU=1450

wget https://raw.githubusercontent.com/onap/vnfsdk-refrepo/master/vnfmarket-be/deployment/install/docker-compose.yml

docker-compose up -d


Install VTP CLI
~~~~~~~~~~~~~~~~

export CLI_TAG=2.0.5-STAGING-20181227T184001Z

docker pull ${NEXUS_DOCKER_REPO}/onap/cli:${CLI_TAG}

docker run -d -e OPEN_CLI_MODE=daemon -p 30260:80 -p 30271:8080 --name vtp-cli ${NEXUS_DOCKER_REPO}/onap/cli:${CLI_TAG}


VERIFY deployment
~~~~~~~~~~~~~~~~~~~~~

docker exec vtp-cli  bash -c "OPEN_CLI_PRODUCT_IN_USE=onap-vtp oclip vnftest-list --host-url http://<docker host ip>:8702"

+----------------+------------------------+
|tescase         |yaml                    |
+----------------+------------------------+
|csar-validate   |vtp-validate-csar.yaml  |
+----------------+------------------------+


NOTE: if failed to run, then follow below guidelines

    docker exec -it refrepo  bash

    export OPEN_CLI_HOME=/opt/vtp

    cd $OPEN_CLI_HOME/bin

    ./oclip-grpc-server.sh &

    Exit docker by running CTRL+p+q

    Run the vnftest-list again as above and verity that its working !!


Setup sample csars
~~~~~~~~~~~~~~~~~~~

Download the sample CSAR to VTP docker from github

    docker exec -it refrepo  bash

    echo nameserver 8.8.8.8 >> /etc/resolv.conf

    cd /opt/vtp

    mkdir csar

    cd csar

    wget https://github.com/onap/vnfsdk-validation/raw/master/csarvalidation/src/test/resources/VoLTE.csar

    exit

NOTE: To test vendor specific CSAR, copy them to /opt/vtp/csar folder using 'docker cp xxx.csar refrepo:/opt/vtp/csar' command


Run the validate csar test case
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

docker exec vtp-cli  bash -c "OPEN_CLI_PRODUCT_IN_USE=onap-vtp BUILD_TAG=demo oclip vnftest-run --name csar-validate --param csar=/opt/vtp/csar/VoLTE.csar --host-url http://<docker host ip>:8702"

+------------+------------------+
|property    |value             |
+------------+------------------+
|results     |{error=SUCCESS}   |
+------------+------------------+
|build_tag   |demo              |
+------------+------------------+
|criteria    |PASS              |
+------------+------------------+


CVC Structures
==============

|image2|

.. |image2| image:: cvc.png
   :height: 600px
   :width: 600px


Casablanca Implemented requriements
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

   * - R-43958
     - The VNF Package MUST include documentation describing the tests that were conducted by the VNF provider and the test results.

   * - R-66070
     - The VNF Package MUST include VNF Identification Data to uniquely identify the resource for a given VNF provider. The identification data must include: an identifier for the VNF, the name of the VNF as was given by the VNF provider, VNF description, VNF provider, and version.

   * - R-77707
     - The VNF provider MUST include a Manifest File that contains a list of all the components in the VNF package.

   * - R-77786
     - The VNF Package MUST include all relevant cookbooks to be loaded on the ONAP Chef Server.
