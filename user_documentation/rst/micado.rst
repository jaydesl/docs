.. _micado_client_lib:

MiCADO client library
*********************

Overview
--------

MiCADO client library extends the MiCADO functionality with MiCADO master deployment
capabilities and application management. The library aims to provide a basic API from
Python environment and support the following:

* Deploy MiCADO service
    - Create, Destroy MiCADO master VM
* Manage application
    - Create, Update, Delete MiCADO applications

Currently, client library supports only NOVA interface. We plan to extend with additional
interfaces later.

Prerequisites
-------------

Install requirements
~~~~~~~~~~~~~~~~~~~~
The required Python packages are defined under the ``requirements.txt``. Make sure to
install those before using MiCADO client library.

Specify cloud credential
~~~~~~~~~~~~~~~~~~~~~~~~
Specify cloud credential for MiCADO master VM creation. The home directory located under

.. code:: yaml

    resource:
    -
        type: nova
        auth_data:
            # Select your authentication method
            # Option #1
            username:
            password:
            # Option #2
            application_credential_id:
            application_credential_secret:


Example
-------

**Usage with a launcher:**

.. note::
    Before you start testing, make sure the authentication data in the correct place.

.. code:: Python

    from micado import MicadoClient

    client = MicadoClient(launcher="openstack")
    client.master.create(auth_url='yourendpoint',
                         project_id='project_id',
                         image='image_name or image_id',
                         flavor='flavor_name or flavor_id',
                         network='network_name or network_id',
                         keypair='keypair_name or keypair_id',
                         security_group='security_group_name or security_group_id')
    client.applications.list()
    client.master.destroy(id='VM ID',
                          auth_url='yourendpoint',
                          project_id='project_id')



**Usage with a launcher:**

.. code:: Python

    from micado import MicadoClient

    client = MicadoClient(endpoint="https://micado/toscasubmitter/",
                          version="v2.0",
                          verify=False,
                          auth=("ssl_user", "ssl_pass"))
    client.applications.list()

Planned features:
-----------------
* Handle multiple MiCADO-master VM
* Support additional Cloud interface


micado package
==============

Subpackages
-----------

.. toctree::
   :maxdepth: 4

   micado.api
   micado.models
   micado.types
   micado.utils

Submodules
----------

micado.client module
--------------------

.. automodule:: micado.client
   :members:
   :undoc-members:
   :show-inheritance:

micado.exceptions module
------------------------

.. automodule:: micado.exceptions
   :members:
   :undoc-members:
   :show-inheritance:

Module contents
---------------

.. automodule:: micado
   :members:
   :undoc-members:
   :show-inheritance:
