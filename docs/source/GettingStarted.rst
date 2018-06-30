Getting started
================

For this developer release, you can get started with the `Enigma Docker Network <https://github.com/enigmampc/enigma-docker-network>`_. This containerized environment will provide you with a sandbox for writing secret contracts. Before you begin, ensure that your computer meets the minimum requirements for this release.

Requirements
~~~~~~~~~~~~

To begin working with this developer release, please ensure you have the
following:

-  A host machine with Intel `Software Guard Extensions <https://software.intel.com/en-us/sgx>`_ (SGX) enabled.
   
      The `SGX hardware <https://github.com/ayeks/SGX-hardware>`_ repository provides a list of hardware that supports Intel SGX, as well as a simple script to check if SGX is enabled on your system.

-  A host machine running Ubuntu 16.04

-  A host machine with `Linux SGX driver <https://github.com/intel/linux-sgx-driver>`_ installed. Upon successful installation of the driver, /dev/isgx should be present in the system.

-  `Docker <https://docs.docker.com/install/overview/>`_

-  `Docker Compose <https://docs.docker.com/compose/install/>`_

Your First Secret Contract
~~~~~~~~~~~~~~~~~~~~~~~~~~

For now, the most up-to-date instructions can be found in the Readme for `Enigma Docker Network <https://github.com/enigmampc/enigma-docker-network>`_.
Coming soon: A tutorial on writing a secret contract.