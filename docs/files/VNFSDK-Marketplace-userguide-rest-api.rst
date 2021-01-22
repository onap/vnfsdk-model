.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2020 Nokia.

VNF SDK Marketplace User Guide for Rest API
============================================

The VNF SDK Marketplace expose rest API endpoints that allows validation of CSAR packages.

**Parameters that need to be inserted are surrounded with {...}**.

- CSAR Validation - use all rules

Send and validate CSAR, against all active rules.
If validation finish before timeout, result will be returned in json format.
Otherwise *executionId*, that can be used for checking validation state in the future, will be returned.

.. code-block::

    curl --location --request POST 'http://{marketplace address}/onapapi/vnfsdk-marketplace/v1/vtp/executions' \
    --header 'Content-Type: multipart/form-data' \
    --form 'file=@"{path to csar file}"' \
    --form 'executions="[{
      \"scenario\": \"onap-vtp\",
      \"testSuiteName\": \"validation\",
      \"testCaseName\": \"csar-validate\",
      \"parameters\": {
        \"csar\": \"file://{csar file name}\",
        \"pnf\":\"true\"
      }
    }]"'


- CSAR Validation - use selected rules

Send and validate CSAR, against selected rules.
If validation finish before timeout, result will be returned in json format.
Otherwise *executionId*, that can be used for checking validation state in the future, will be returned.

.. code-block::

    curl --location --request POST 'http://{marketplace address}/onapapi/vnfsdk-marketplace/v1/vtp/executions' \
    --header 'Content-Type: multipart/form-data' \
    --form 'file=@"{path to csar file}"' \
    --form 'executions="[{
      \"scenario\": \"onap-vtp\",
      \"testSuiteName\": \"validation\",
      \"testCaseName\": \"csar-validate\",
      \"parameters\": {
        \"csar\": \"file://{csar file name}\",
        \"pnf\":\"true\",
        \"rules\":\"{rules to be validated example:r130206,r816745}\"
      }
    }]"'

- CSAR Validation - run validation for selected release

Send and validate a CSAR file using rules which have selected release.
The 'release' parameter is optional and it can have one of the values: amsterdam, casablanca, dublin, frankfurt, guilin, honolulu, latest
If user doesn't set parameter then all rules will be used during validation. Otherwise rules are collected according to pattern:
amsterdam rules <- casablanca rules <-dublin rules <-frankfurt rules <- guilin rules <- honolulu release <- latest
For example: if user set release to dublin then rules with release dublin, casablanca and amsterdam will be used.

If validation finish before timeout, result will be returned in json format.
Otherwise *executionId*, that can be used for checking validation state in the future, will be returned.

.. code-block::

    curl --location --request POST 'http://{marketplace address}/onapapi/vnfsdk-marketplace/v1/vtp/executions' \
    --header 'Content-Type: multipart/form-data' \
    --form 'file=@"{path to csar file}"' \
    --form 'executions="[{
      \"scenario\": \"onap-vtp\",
      \"testSuiteName\": \"validation\",
      \"testCaseName\": \"csar-validate\",
      \"parameters\": {
        \"csar\": \"file://{csar file name}\",
        \"pnf\":\"true\",
        \"release\":\"dublin\"
      }
    }]"'

- CSAR Validation - get results of validation

Get result of CSAR validation connected with *executionId*.
Returns result in json format.

.. code-block::

    curl --location --request GET 'http://{marketplace address}/onapapi/vnfsdk-marketplace/v1/vtp/executions?requestId={executionId}'



- Example

Request:

.. code-block::

    curl --location --request POST 'http://127.0.0.1:8702/onapapi/vnfsdk-marketplace/v1/vtp/executions' \
    --header 'Content-Type: multipart/form-data' \
    --form 'file=@"/csar/invalidSignaturePackage.csar"' \
    --form 'executions="[{
      \"scenario\": \"onap-vtp\",
      \"testSuiteName\": \"validation\",
      \"testCaseName\": \"csar-validate\",
      \"parameters\": {
        \"csar\": \"file://invalidSignaturePackage.csar\",
        \"pnf\":\"true\",
        \"rules\":\"r130206,r816745\"
      }
    }]"'


Response:

.. code-block::

    [
        {
            "scenario": "onap-vtp",
            "testCaseName": "csar-validate",
            "testSuiteName": "validation",
            "executionId": "5e7a1726-4c48-42b9-ade4-dfd12ea75107-1608035294086",
            "parameters": {
                "csar": "/tmp/data/vtp-tmp-files/invalidSignaturePackage.csar",
                "pnf": "true",
                "rules": "r130206,r816745"
            },
            "results": {
                "vnf": {
                    "name": "RadioNode",
                    "vendor": "Ericsson",
                    "version": "1.0",
                    "type": "TOSCA",
                    "mode": "WITH_TOSCA_META_DIR"
                },
                "date": "Tue Dec 15 12:28:14 UTC 2020",
                "criteria": "FAILED",
                "results": [
                    {
                        "passed": true,
                        "vnfreqName": "SOL004",
                        "description": "V2.4.1 (2018-02)",
                        "errors": [],
                        "warnings": []
                    },
                    {
                        "passed": false,
                        "vnfreqName": "r130206",
                        "description": "The VNF/PNF package shall contain a Digest (a.k.a. hash) for each of the components of the VNF package. The table of hashes is included in the manifest file, which is signed with the VNF provider private key. In addition, the VNF provider shall include a signing certificate that includes the VNF provider public key, following a pre-defined naming convention and located either at the root of the archive or in a predefined location (e.g. directory).",
                        "errors": [
                            {
                                "vnfreqNo": "R130206",
                                "code": "0x4007",
                                "message": "File has invalid signature!",
                                "lineNumber": -1
                            }
                        ],
                        "warnings": []
                    },
                    {
                        "passed": false,
                        "vnfreqName": "r816745",
                        "description": "The VNF or PNF PROVIDER MUST provide the Service Provider with PM Meta Data (PM Dictionary)\nto support the analysis of PM events delivered to DCAE.\nThe PM Dictionary is to be provided as a separate YAML artifact at onboarding and must follow\nthe VES Event Listener Specification and VES Event Registration Specification\nwhich contain the format and content required.",
                        "errors": [
                            {
                                "vnfreqNo": "R816745",
                                "code": "0x2000",
                                "message": "Fail to load PM_Dictionary With error: PM_Dictionary YAML file is empty",
                                "file": "Artifacts/Deployment/Measurements/PM_Dictionary.yml",
                                "lineNumber": -1
                            }
                        ],
                        "warnings": []
                    }
                ],
                "contact": "ONAP VTP Team onap-discuss@lists.onap.org",
                "platform": "VNFSDK - VNF Test Platform (VTP) 1.0"
            },
            "status": "COMPLETED",
            "startTime": "2020-12-15T12:28:11.895",
            "endTime": "2020-12-15T12:28:14.962"
        }
    ]
