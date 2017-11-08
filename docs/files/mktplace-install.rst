VNF SDK Marketplace Installation Instructions
============================================

1. Download vnfsdk/refrepo from Gerrit
::

  git clone http://gerrit.onap.org/r/vnfsdk/refrepo

2. Goto vnfmarket-be/deployment/install,
::

  Enter command "docker-compose up -d"

This will start two Docker containers:

    a. VNF Repository
    b. PostgreSQL database.

Once started, access the Marketplace from your web browser.

3. Connect to http://{host}:8702/openoui/vnfmarket to access the user interface
