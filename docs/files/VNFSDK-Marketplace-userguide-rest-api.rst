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
      \"scenario\": \"onap-dublin\",
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
      \"scenario\": \"onap-dublin\",
      \"testSuiteName\": \"validation\",
      \"testCaseName\": \"csar-validate\",
      \"parameters\": {
        \"csar\": \"file://{csar file name}\",
        \"pnf\":\"true\",
        \"rules\":\"{rules to be validated example:r130206,r816745}\"
      }
    }]"'


- CSAR Validation - get results of validation

Get result of CSAR validation connected with *executionId*.
Returns result in json format.

.. code-block::

    curl --location --request GET 'http://{marketplace address}/onapapi/vnfsdk-marketplace/v1/vtp/executions?requestId={executionId}'
