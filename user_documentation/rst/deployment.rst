.. _deployment:

Deployment
**********

To deploy MiCADO you need a (separate) virtual machine, called MiCADO master. There are two ways of deployment:

* remote: download the Ansible playbook on your local machine, configure the MiCADO master as target machine and run the playbook to perform the deployment remotely.
* local: login to the MiCADO master, download the Ansible playbook, configure the localhost as target machine and run the playbook to perform the deployment locally.

We recommend to perform the installation remotely as all your configuration files are preserved on your machine, i.e. it is easier to repeat the deployment if needed.

Prerequisites
=============

For cloud interfaces supported by MiCADO:

* EC2 (tested on Amazon and OpenNebula)
* Nova (tested on OpenStack)
* CloudSigma
* CloudBroker

For the MiCADO master:

* Ubuntu 16.04

For the host where the Ansible playbook is executed (differs depending on local or remote):

* Ansible 2.4 or greater
* Git


Ansible
-------

Note: Ansible in the Ubuntu 16.04 APT repository is outdated and insufficient (at the time of writing this document)

To install Ansible on Ubuntu 16.04, use these commands:

::

   sudo apt-get update
   sudo apt-get install software-properties-common
   sudo apt-add-repository ppa:ansible/ansible
   sudo apt-get update
   sudo apt-get install ansible

To install Ansible on other operation systems follow the `official installation guide <https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html>`__.

Git
---

To install Git on Ubuntu, use this command:

::

   sudo apt-get install git-all

To install Git on other operating systems follow the `official installation guide <https://git-scm.com/book/en/v2/Getting-Started-Installing-Git>`__.

Installation
============

Perform the following steps either on your local machine or on MiCADO master depending on the installation method.

Step 1: Download the ansible playbook.
--------------------------------------

::

   git clone https://github.com/micado-scale/ansible-micado.git ansible-micado
   cd ansible-micado
   git checkout v0.6.1

Step 2: Specify cloud credential for instantiating MiCADO workers.
------------------------------------------------------------------

MiCADO master will use this credential to start/stop VM instances (MiCADO workers) to host the application and to realize scaling. Credentials here should belong to the same cloud as where MiCADO master is running. We recommend making a copy of our predefined template and edit it. MiCADO expects the credential in a file, called credentials-cloud-api.yml before deployment. Please, do not modify the structure of the template!

::

   cp sample-credentials-cloud-api.yml credentials-cloud-api.yml
   edit credentials-cloud-api.yml

Edit credentials-cloud-api.yml to add cloud credentials. You will find predefined sections in the template for each cloud interface type MiCADO supports. Fill only the section belonging to your target cloud.

Optionally you can use the `Ansible Vault <https://docs.ansible.com/ansible/2.4/vault.html>`_ mechanism to keep the credential data in an encrypted format. To achieve this, create the above file using Vault with the command

::

    ansible-vault create credentials-cloud-api.yml


This will launch the editor defined in the ``$EDITOR`` environment variable to make changes to the file. If you wish to make any changes to the previously encrypted file, you can use the command

::

    ansible-vault edit credentials-cloud-api.yml

Step 3a: Specify security settings and credentials to access MiCADO.
--------------------------------------------------------------------

MiCADO master will use these security-related settings and credentials to authenticate its users for accessing the REST API and Dashboard.

::

   cp sample-credentials-micado.yml credentials-micado.yml
   edit credentials-micado.yml

Specify the provisioning method for the x509 keypair used for TLS encryption of the management interface in the ``tls`` subtree:

* The 'self-signed' option generates a new keypair with the specified hostname as subject (or 'micado-master' if omitted).
* The 'user-supplied' option lets the user add the keypair as plain multiline strings (in unencrypted format) in the ansible_user_data.yml file under the 'cert' and 'key' subkeys respectively.

Specify the default username and password for the administrative we user in the the ``authentication`` subtree.

Optionally you may use the Ansible Vault mechanism as described in Step 2 to protect the confidentiality and integrity of this file as well.


Step 3b: (Optional) Specify credentials to use private Docker registries.
-------------------------------------------------------------------------

Set the Docker login credentials of your private Docker registry in which your private containers are stored. We recommend making a copy of our predefined template and edit it. MiCADO expects the docker registry credentials in a file, called credentials-docker-registry.yml. Please, do not modify the structure of the template!

::

   cp sample-credentials-docker-registry.yml credentials-docker-registry.yml
   edit credentials-docker-registry.yml

Edit credentials-docker-registry.yml and add username, password, and registry url. To login to the default docker_hub, leave DOCKER_REPO as is (a blank string).

Optionally you may use the Ansible Vault mechanism as described in Step 2 to protect the confidentiality and integrity of this file as well.

Step 4: Launch an empty cloud VM instance for MiCADO master.
------------------------------------------------------------

This new VM will host the MiCADO core services. 

**a)** Default port number for MiCADO service is ``443``. Optionally, you can modify the port number stored by the variable called ``web_listening_port`` defined in the ansible playbook file called ``micado-master.yml``. 

**b)** Configure a cloud firewall settings which opens the following ports on the MiCADO master virtual machine:

::

   TCP: 22,2377,7946,8300,8301,8302,8500,8600,[web_listening_port]
   UDP: 4789,7946,8301,8302,8600

**NOTE:** replace ``[web_listening_port]`` with the actual value specified in Step 4a.

**NOTE:** MiCADO master has built-in firewall, therefore you can leave all ports open at cloud level.

**c)** Finally, launch the virtual machine with the proper settings (capacity, ssh keys, firewall): use any of aws, ec2, nova, etc command-line tools or web interface of your target cloud to launch a new VM. We recommend a VM with 2 cores, 4GB RAM, 20GB disk. Make sure you can ssh to it (password-free i.e. ssh public key is deployed) and your user is able to sudo (to install MiCADO as root). Store its IP address which will be referred as ``IP`` in the following steps.

Step 5: Customize the inventory file for the MiCADO master.
-----------------------------------------------------------

We recommend making a copy of our predefined template and edit it. Use the template inventory file, called sample-hosts for customisation.

::

   cp sample-hosts hosts
   edit hosts

Edit the ``hosts`` file to set ansible variables for MiCADO master machine. Update the following parameters:

* **ansible_host**: specifies the publicly reachable ip address of MiCADO master. Set the public or floating ip of the master regardless the deployment method is remote or local. The ip specified here is used by the Dashboard for webpage redirection as well
* **ansible_connection**: specifies how the target host can be reached. Use "ssh" for remote or "local" for local installation. In case of remote installation, make sure you can authenticate yourself against MiCADO master. We recommend to deploy your public ssh key on MiCADO master before starting the deployment
* **ansible_user**: specifies the name of your sudoer account, defaults to "ubuntu"
* **ansible_become**: specifies if account change is needed to become root, defaults to "True"
* **ansible_become_method**: specifies which command to use to become superuser, defaults to "sudo"
* **ansible_python_interpreter**: specifies the interpreter to be used for ansible on the target host, defaults to "/usr/bin/python3"

Please, revise all the parameters, however in most cases the default values are correct.

Step 6: Start the installation of MiCADO master.
------------------------------------------------

::

   ansible-playbook -i hosts micado-master.yml

If you have used Vault to encrypt your credentials, you have to add the path to your vault credentials to the command line as described in the `Ansible Vault documentation <https://docs.ansible.com/ansible/2.4/vault.html#providing-vault-passwords>`_ or provide it via command line using the command
::

    ansible-playbook -i hosts micado-master.yml --ask-vault-pass

After deployment
================

Once the deployment has successfully finished, you can proceed with

* visiting the :ref:`dashboard`
* using the :ref:`restapi`
* playing with the :ref:`tutorials`
* creating your :ref:`applicationdescription`

Check the logs
==============

You can SSH into MiCADO master and check the logs at any point after MiCADO is succesfully deployed. All logs are kept under ``/var/log/micado`` and are organised by components. Scaling decisions, for example, can be inspected under ``/var/log/micado/policykeeper``

Accessing user-defined service
==============================

In case your application contains container exposing a service, you have two alternatives to access its endpoint:

* via MiCADO master: open up your service port number on the MiCADO master's internal firewall before deployment. To do that, extend the firewall configuration by editing the file(s) located at in the ``roles/micado-master/templates/iptables`` directory. Make sure you open up the cloud firewall as well for the MiCADO master!

* via MiCADO worker: query the ip address of the worker nodes. You can do that through the Dashboard of MiCADO, the Dashboard of your cloud or the REST API of MiCADO. Make sure the port of your service is open up by the cloud firewall for the MiCADO workers!
