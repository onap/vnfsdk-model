.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. http://creativecommons.org/licenses/by/4.0
.. Copyright 2020 Nokia

VNF SDK OOM Installation
============================================

How to install VNF SDK from OOM:
--------------------------------

1. Uninstall all components
~~~~~~~~~~~~~~~~~~~~~~~~~~~

    helm undeploy dev --purge

2. Clear all what have left
~~~~~~~~~~~~~~~~~~~~~~~~~~~

    for resource in deployment statefulset job pod pvc pv; do kubectl -n onap delete $resource --all --force --grace-period=0 & done

    cd /dockerdata-nfs/dev/
    sudo rm -rf *

3. Remove namespace
~~~~~~~~~~~~~~~~~~~

    kubectl delete namespace onap

4. After changes in component in directory oom/kubernetes execute:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    make vnfsdk

5. Then you have to make entire ONAP project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    make onap -e SKIP_LINT=TRUE

6. Install ONAP on lab:
~~~~~~~~~~~~~~~~~~~~~~~

    if there is file deploy-onap.sh in directory oom/kubernetes

    ./deploy-onap.sh

    otherwise:

    helm deploy dev local/onap --namespace onap -f onap/resources/overrides/onap-all.yaml -f ./openstack-30-elalto.yaml  --timeout 1000 --verbose 2>&1 | tee ~/helm-installation-manual.log
