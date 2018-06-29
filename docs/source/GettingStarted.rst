.. _getting-started:

Getting started
================

For this developer release, you can get started with the Enigma Docker
Network [UPDATE WITH LINK TO PUBLIC REPO]. This containerized
environment will provide you with a sandbox for writing secret
contracts.

Requirements
~~~~~~~~~~~~

To begin working with this developer release, please ensure you have the
following:

-  A host machine with Intel `Software Guard
      Extensions <https://software.intel.com/en-us/sgx>`__ (SGX)
      enabled.

-  A host machine with Intel `Software Guard
      Extensions <https://software.intel.com/en-us/sgx>`__ (SGX)
      enabled.

   -  The `SGX hardware <https://github.com/ayeks/SGX-hardware>`__
         repository provides a list of hardware that supports Intel SGX,
         as well as a simple script to check if SGX is enabled on your
         system.

-  A host machine running Ubuntu 16.04

-  A host machine with `Linux SGX
      driver <https://github.com/intel/linux-sgx-driver>`__ installed.
      Upon successful installation of the driver /dev/isgx should be
      present in the system.

-  Docker

-  `Docker Compose <https://docs.docker.com/compose/install/>`__

Your First Secret Contract
~~~~~~~~~~~~~~~~~~~~~~~~~~

This should contain the info from USAGE and TESTING in addition to a
walkthrough for a sample contract.

NOTE FROM MOIRA, UNSURE OF PLACEMENT: It is very important that in the
solidity code the parameters in callback and the return parameters for
callable should be exactly (also syntactically) the same. Otherwise
there will be problems with the hash of the functions.