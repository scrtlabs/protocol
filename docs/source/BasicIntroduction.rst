
Basic Introduction
==================

This page contains a lightly-worded and easily digestable overview of both 
the Enigma protocol and this Docker testnet release. For more thorough 
and indepth information, see the 
:doc:`technical introduction <TechnicalIntroduction>` or the various 
architecture pages.

How Enigma Works
~~~~~~~~~~~~~~~~~

The Enigma protocol is a decentralized and distributed network of servers 
around the world (known as 'nodes') which, through the use of 
`secret contracts <https://blog.enigma.co/defining-secret-contracts-f40ddee67ef2>`__,
are able to compute data in a way that maintains confidentiality and integrity. 
Secret contracts ensure that the data is kept verifiably untampered and private 
from the beginning to the end of the process, including from the node performing 
the computational task. The Enigma Network is also able to take on task orders from
Ethereum.


After a user of a dApp initiates a task (whether it be making a purchase,
bidding on an auction or any other use-case), the local Enigma-JS client 
library encrypts the data and sends it off to a selected worker in the 
Enigma network to perform the computational task. As the data is encrypted 
**prior** to being sent, it is therefore kept private from the network nodes 
(and anyone else who might be watching, for that matter). After completion
the answer is securely passed back to the end-user. 

This order of operations can be thought of in five steps:

1) A dApp end-user initiates a task. Data passed to Enigma-JS.
2) Data is encrypted by the Enigma-JS client library.
3) The now encrypted data is sent off to the Enigma Network.
4) Task is broadcasted to the network. A worker is selected to compute the task.
5) Computation is performed, node is rewarded and answer to task is returned.

For more information on this process as well as some helpful visuals, 
check out the :doc:`software architecture <SoftwareArchitecture>` page.

The Enigma Testnet - What's Inside?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This testnet release is an entirely self-contained network (deployed using 
`Docker <https://docker.com>`__) which is intended to let developers have
a chance to build their first secret contracts and learn how Enigma's
components operate. As the Enigma Network is not live yet, this release
contains a local version of every component necessary for a fully operating 
network, nodes and all.

Inside the 
`Enigma Docker Network <https://github.com/enigmampc/enigma-docker-network>`__
is four independent containers - a **principal node**, two **standard nodes**
and one for the blockchain logic itself. By following along with the 
:doc:`getting started <GettingStarted>` guide down to the 
:doc:`creating a React front-end <CreateReactFrontend>` section, you will
gain an understanding of how these individual containers as well as Enigma's 
core components interact with eachother.

For a more thorough breakdown of this process including some helpful visuals, 
see the :doc:`network topology <NetworkTopology>` page.
