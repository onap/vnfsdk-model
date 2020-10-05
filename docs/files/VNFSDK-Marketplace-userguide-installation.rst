.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2020 Nokia

VNF SDK OOM Installation
========================


Prerequisite
------------

    All operations must be executed at RKE - log in at RKE.

How to install ONAP from OOM:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**1. Uninstall all components**

    helm undeploy dev --purge

    TIP: Operation takes about 30-60 minutes. Please wait until it ends.

**2. Clear all what have left**

    Execute below command:

    for resource in deployment statefulset job pod pvc pv; do kubectl -n onap delete $resource --all --force --grace-period=0 & done

    cd /dockerdata-nfs/dev/
    sudo rm -rf *

**3. Remove namespace**

    kubectl delete namespace onap

**4. After changes in component in directory oom/kubernetes execute:**

    make vnfsdk

    NOTE:
    This folder is in /home/ubuntu folder
    If you need to change a refrepo image or do other modifications, you need to make changes in values.yaml. Mentioned file is in /home/ubuntu/oom/kubernetes/vnfsdk.

**5. Then you have to make entire ONAP project**

    make onap -e SKIP_LINT=TRUE

**6. Install ONAP on lab:**

    If there is file deploy-onap.sh in directory oom/kubernetes, pls execute below command.

    ./deploy-onap.sh

    otherwise:

    helm deploy dev local/onap --namespace onap -f onap/resources/overrides/onap-all.yaml -f ./openstack-30-elalto.yaml  --timeout 1000 --verbose 2>&1 | tee ~/helm-installation-manual.log


How to install VNF SDK from OOM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**1. Uninstall VNF SDK Component**

    helm undeploy dev-vnfsdk  --purge

**2. Clear all what have left**

    sudo rm -rf /dockerdata-nfs/dev/vnfsdk/

**3. After changes in component in directory oom/kubernetes execute:**

    make vnfsdk

    NOTE:
    This folder is in /home/ubuntu folder
    If you need to change a refrepo image or do other modifications, you need to make changes in values.yaml. Mentioned file is in /home/ubuntu/oom/kubernetes/vnfsdk.

**4. Install ONAP on lab**

    helm deploy dev-vnfsdk local/onap --namespace onap -f onap/resources/overrides/onap-all.yaml -f ./openstack-30-elalto.yaml  --timeout 1000 --verbose 2>&1 | tee ~/helm-installation-manual.log


How to upgrade Refrepo in VNF SDK from OOM:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


**1. Edit Refrepo image in VNF SDK Ddeployment**

     kubectl -n onap edit deployment dev-vnfsdk

    then change:

.. code-block:: yaml

        image: nexus.onap.dyn.nesc.nokia.net:10001/onap/vnfsdk/refrepo:1.6.0-STAGING-latest
        imagePullPolicy: Always
        name: vnfsdk

**2. Check if VNF SDK works**

    curl --insecure -i -H "Accept: application/json" -H "Content-Type: application/json" -X GET https://WORKER_IP:REFREPO_PORT/onapapi/vnfsdk-marketplace/v1/vtp/scenarios

    NOTE: To get information about REFREPO_PORT, pls execute:

        kubectl -n onap get service | grep refrepo

**3. Enter at vnfsdkmarketplace UI**

    In your browser go to: http://WORKER_IP:REFREPO_PORT/onapui/vnfmarket



Getting into VNF SDK pod
~~~~~~~~~~~~~~~~~~~~~~~~


**1. Enter the VNF SDK pod**

    kubectl -n onap exec -it <vnfsdk pod> /bin/bash

    NOTE:
    To find <vnfsdk  pod> execute: kubectl -n onap get pod | grep vnfsdk

**2. To check logs go to /service/logs**

    cd /service/logs

    and then you can see vnfsdkmarketplace logs executing:

    cat vnfsdkmarketplace.log

    or catalina logs executing:

    cat catalina.out