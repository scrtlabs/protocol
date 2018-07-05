Getting started
================

For this developer release, you can get started with the 
`Enigma Docker Network <https://github.com/enigmampc/enigma-docker-network>`_. 
This containerized environment will provide you with a sandbox for writing 
secret contracts. Before you begin, ensure that your computer meets the minimum 
requirements for this release.

Hardware Requirements
~~~~~~~~~~~~~~~~~~~~~

To begin working with this developer release, you will need a host machine with 
Intel `Software Guard Extensions <https://software.intel.com/en-us/sgx>`_ (SGX) 
enabled.
   
The `SGX hardware <https://github.com/ayeks/SGX-hardware>`_ repository provides
a list of hardware that supports Intel SGX, as well as a simple script to check 
if SGX is enabled on your system.

Generally speaking you need a computer from 2015 or newer, with an Intel CPU. 
You may be able to procure a desktop or a laptop (refer to the list above, 
HP or Dell brands seem to be the safer bet, whereas anything with MacOS is not 
supported). 

Alternatively, you may consider cloud infrastructure, where the only option 
(as of July 2018) is 
`IBM bare metal servers <https://www.ibm.com/cloud/bare-metal-servers>`_ at
a cost of about $275/month (refer to 
`this post <https://github.com/ayeks/SGX-hardware/issues/43>`_ for the specs of 
the server to provision with SGX capabilities). Amazon Web Services, Google 
Cloud and Microsoft Azure are known not to support SGX.

Software Requirements
~~~~~~~~~~~~~~~~~~~~~

-  A host machine with 
   `Linux SGX driver <https://github.com/intel/linux-sgx-driver>`_ installed. 
   Upon successful installation of the driver, ``/dev/isgx`` should be present 
   in the system.

-  `Docker <https://docs.docker.com/install/overview/>`_

-  `Docker Compose <https://docs.docker.com/compose/install/>`_

Your First Secret Contract
~~~~~~~~~~~~~~~~~~~~~~~~~~

For now, the most up-to-date instructions can be found in the Readme for 
`Enigma Docker Network <https://github.com/enigmampc/enigma-docker-network>`_.
Coming soon: A tutorial on writing a secret contract.