
Writing a Secret Contract
=========================

Secret contracts are smart contracts that execute code over Enigma's decentralized 
network without revealing input data, ensuring both integrity and confidentiality. Secret 
contracts are written in a programming language called Solidity, and are implemented 
very similarly to Ethereum-based smart contracts.

**NOTICE:** Although the ``enigma-docker-network`` must stay running, at this point in the 
guide you will be operating entirely out of the 
``enigma-template-dapp`` folder (the ``dapp`` terminal). 

Example 01: The Millionaires Problem
``````````````````````````````````

The millionaires problem is a simple concept with complex solutions: How do two 
individuals compare their net worth without ever exposing their actual value to 
the other party?

Inside your ``dapp`` terminal, enter the ``/contracts`` folder and create a new 
file called ``MillionairesProblem.sol`` and paste the following contract code within
it: ::

 pragma solidity ^0.4.17; 


 contract MillionairesProblem {
	// Stores # of millionaires that have joined and stated their net worths
	uint public numMillionaires; 
	// Stores Millionaire structs (defined below)
	Millionaire[] millionaires; 
	// Stores address of richest millionaire; set in callback function below
	address public richestMillionaire; 
	address public owner;
	address public enigma;

	// Millionaire struct which holds encrypted address and net worth
	struct Millionaire {
		bytes myAddress; 
		bytes myNetWorth; 
	}

	// Event emitted upon callback completion; watched from front end
	event CallbackFinished(); 

	// Modifier to ensure only enigma contract can call function
	modifier onlyEnigma() {
		require(msg.sender == enigma);
		_;
	}

	// Constructor called when new contract is deployed
	constructor(address _enigmaAddress, address _owner) public {
		owner = _owner; 
		enigma = _enigmaAddress;
	}

	// Add a new millionaire with encrypted address and net worth arguments
	function addMillionaire(bytes _encryptedAddress, bytes _encryptedNetWorth) 
		public 
	{
		Millionaire memory millionaire = Millionaire({
			myAddress: _encryptedAddress, 
			myNetWorth: _encryptedNetWorth
		}); 
		millionaires.push(millionaire); 
		numMillionaires++; 
	}

	// Return encrypted address and net worth for a particular millionaire
	function getInfoForMillionaire(uint index) 
		public 
		view 
		returns (bytes, bytes) 
	{
		Millionaire memory millionaire = millionaires[index]; 
		bytes memory encryptedAddress = millionaire.myAddress; 
		bytes memory encryptedNetWorth = millionaire.myNetWorth; 
		return (encryptedAddress, encryptedNetWorth); 
	}
	
	/*
	CALLABLE FUNCTION run in SGX to decipher encrypted net worths to 
	determine richest millionaire
	*/
	function computeRichest(address[] _addresses, uint[] _netWorths) 
		public 
		pure 
		returns (address) 
	{
		uint maxIndex; 
		uint maxValue; 
		for (uint i = 0; i < _netWorths.length; i++) {
			if (_netWorths[i] >= maxValue) {
				maxValue = _netWorths[i]; 
				maxIndex = i; 
			}
		}
		return _addresses[maxIndex]; 
	}

	/*
	CALLBACK FUNCTION to change contract state tracking richest 
	millionaire's name
	*/
	function setRichestAddress(address _address) public onlyEnigma() {
		richestMillionaire = _address; 
		emit CallbackFinished(); 
	}
 }

As mentioned above, the design principles and syntax (state variables, structs, constructors, 
functions, events, modifiers, etc.) are very similar to Ethereum smart contracts, the two major 
differences being the ``callable`` and ``callback`` functions.

Callable
  This is a public function that runs secret computations inside the SGX enclave. It is a ``pure``
  function, meaning it does not read from nor write to the contract state, but computes solely
  off of the arguments that are passed to it. Although these encrypted values are passed via  
  the front-end interface, the decryption automatically occurs within this function.
  In the case of this ``computeRichest`` callable example, the arguments take the form 
  of ``_addresses`` and ``_netWorths`` - more specifically, types ``address[]`` and ``uint[]``. 
  These arguments are decrypted automatically, and it is now possible to determine the party with
  the highest net worth and retrieve the decrypted address at the same index. This decrypted 
  address is to be used as the input for the ``callback`` function.

Callback 
  This is a public function automatically called by the worker (the ``onlyEnigma()`` modifier) 
  after the callable function is completed. It is responsible for committing the results and 
  altering the contract state. In this example, we input the ``_address`` weâ€™ve obtained from the 
  ``callable``, store it as the ``richestMillionaire`` state variable, and emit the 
  ``CallbackFinished`` event. The output of the final event is important, as it is possible to set
  up an event watcher within the front-end to perform a task upon successful completion.

The next step is to create a contract 'factory design pattern' so fresh instances of the 
``MillionairesProblem`` can be generated on-demand. From the ``/contracts`` folder, create a new 
file called ``MillionairesProblemFactory.sol`` and paste the following code: ::

 pragma solidity ^0.4.17; 
 import "./MillionairesProblem.sol";


 contract MillionairesProblemFactory {
	address public enigmaAddress; 
	// List of addresses for deployed MillionaireProblem instances
	address[] public millionairesProblems; 

	constructor(address _enigmaAddress) public {
		enigmaAddress = _enigmaAddress; 
	}

	// Create a new MillionaireProblem and store address to array
	function createNewMillionairesProblem() public {
		address newMillionairesProblem = new MillionairesProblem(
			enigmaAddress, 
			msg.sender
		);
		millionairesProblems.push(newMillionairesProblem); 
	}

	// Obtain list of all deployed MillionaireProblem instances
	function getMillionairesProblems() public view returns (address[]) {
		return millionairesProblems; 
	}	 
 }

All of the necessary solidity code has been completed at this point. The final step is to
implement a migration script that will deploy the ``MillionairesProblemFactory`` contract.
From within the ``migrations/`` folder, create a script called ``2_deploy_millionaires_problem_factory.js``
and paste the following: ::

 const http = require("http");
 const MillionairesProblemFactory = artifacts.require(
    "MillionairesProblemFactory.sol"
 );

 module.exports = function(deployer) {
    return (
        deployer
            .then(() => {
                return new Promise((resolve, reject) => {
                    /* 
                    Obtain the Enigma contract address hosted at this port
                    upon enigma-docker-network launch
                    */
                    const request = http.get(
                        "http://localhost:8081",
                        response => {
                            if (
                                response.statusCode < 200 ||
                                response.statusCode > 299
                            ) {
                                reject(
                                    new Error(
                                        "Failed to load page, status code: " +
                                            response.statusCode
                                    )
                                );
                            }
                            const body = [];
                            response.on("data", chunk => body.push(chunk));
                            response.on("end", () => resolve(body.join("")));
                        }
                    );
                    request.on("error", err => reject(err));
                });
            })
            // Deploy MillionairesProblemFactory with the Enigma contract address
            .then(enigmaAddress => {
                console.log("Got Enigma Contract address: " + enigmaAddress);
                return deployer.deploy(
                    MillionairesProblemFactory,
                    enigmaAddress
                );
            })
            .catch(err => console.error(err))
    );
 };

The function of this script is to pass the Enigma contract address that was deployed 
when the Docker container was launched ( found at ``http://localhost:8081``), and deploy 
a fresh ``MillionairesProblemFactory`` instance with this address as an argument.

**NOTICE:** To create new contracts or modify existing ones, you must redeploy to the network 
with the following command: ``darq-truffle migrate --reset --network development``

Example 02: Secret Auctions
````````````````````````````
Auction theory is a complex economics field involving significant academic research, and thus 
there are a large variety of auction types which enable different economic and social behaviours. 
To best showcase the value of cryptographic privacy in auctions, the below example outlines a 
simple **sealed-bid auction**, which is a variation that protects the value of bids during the 
bidding process. 
 
**NOTICE:** Several `design choices <#design-considerations>`__ were necessary due to the current 
`testnet limitations <GettingStarted#testnet-limitations>`__ of the Enigma Protocol - though if you have
any suggestions for how to improve the methods used, do `let us know! <https://forum.enigma.cO>`__

How it Works
~~~~~~~~~~~~~

**1.** A user creates a new auction by sending a transaction to an â€œAuction Factoryâ€, 
which acts as a proxy for deploying new auction contracts. 

**2.** The Auction Factory specifies a ERC721 contract which will mint the auction reward.

**3.** A potential bidder stakes Ether in the auction contract and acts as collateral for a potential bid. 

**4.** Users will send encrypted bids to the contract. Anyone can change their bid during the bidding process. 

Breaking Down the Code
~~~~~~~~~~~~~~~~~~~~~~

This section explains the individual logic components of the code. To view the combined full 
source of the below snippets, see
`this repository <https://github.com/enigmampc/secret-contracts/blob/master/contracts/Auction.sol>`__.

The Contract State
^^^^^^^^^^^^^^^^^^^
::

 enum AuctionState { IN_PROGRESS, CALCULATING, COMPLETED }

 event Bid(address bidder);
 event Winner(address winner, uint bidValue);

 struct Bidder {
  bool hasBidded;
  bytes bidValue;
 } 

 address public owner;
 uint public startTime;
 uint public endTime;
 address public winner;
 uint public startingPrice;
 uint public winningPrice;
 mapping(address => Bidder) public bidders;
 mapping(address => uint) public stakeAmounts;
 address[] public bidderAddresses;
 Enigma public enigma;
 EnigmaCollectible public enigmaCollectible;
 AuctionState public state;
 bool public rewardClaimed;

This section of the code defines several functions:

**Enum:** The state of the auction is defined by an enum called ``AuctionState``. "Calculating" 
refers to the period when the Enigma network is determining the winner.

**Events:** ``Bid`` refers to individual bids and ``Winner`` signals the final update of the winner.

**Bidder Struct:** Each address has an associated ``Bidder`` struct which contains a boolean 
determining if they have already bidded and their current encrypted bid.

**State Variables:** Most of the variables are straightforward. Note that ``stakeAmounts`` 
refers to the amount of Ether (in wei) that each address has staked.

The Bidding Process
^^^^^^^^^^^^^^^^^^^^
Bidding begins by users staking Ether in the contract, which acts as a binding commitment 
towards paying their bid value in the case they are victorious. It is assumed that users will 
bid an amount less than their deposit in order to obscure the true value of their bid (See 
`design considerations <#design-considerations>`__). 

**NOTICE:** Users can also increase their stake anytime during the bidding process.

::

 function stake() payable external {
   require(state == AuctionState.IN_PROGRESS);
   stakeAmounts[msg.sender] += msg.value;
 }

The contract will now check if the auction is open for bidding and whether the user has 
enough stake to fulfill the minimum bid value (specified at the creation of the auction). 
If these requirementsare met, a bid can be placed on the contract. 

**NOTICE:** Similar to the staking function, a user can update their bid anytime during 
the bidding process ::

 function bid(bytes _bidValue) external {
   require(now < endTime);  
   require(stakeAmounts[msg.sender] >= startingPrice);  
   bidders[msg.sender].bidValue = _bidValue;
   if (!(bidders[msg.sender].hasBidded)) {
     bidders[msg.sender].hasBidded = true;
   }
   emit Bid(msg.sender);
 }

Finally, the creator of the auction will end the auction when the bidding period expires. ::

 function endAuction() external isOwner {
   require(state == AuctionState.IN_PROGRESS);
   require(now >= endTime);
   state = AuctionState.CALCULATING;
 }

Post-Auction: Callable and Callback
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After the auction has come to an end, the dApp which deployed the contracts 
will submit a task to the Enigma Network in order to calculate the winner. 
This is where the standard ``callable`` and ``callback`` functions that are 
fundamental to the privacy-preserving nature of Enigma are utilized. ::

  /*
   * The callable function. Gets the highest bidder and bid amount for the auction.
   */

  function getHighestBidder(address[] _bidders, uint[] _bidAmounts, uint[] _stakeAmounts) public pure returns (address, uint) {
    address highestBidder;
    uint highestBidAmount;
    for (uint i = 0; i < _bidders.length; i++) {
      if ((_bidAmounts[i] > highestBidAmount) && (_bidAmounts[i] <= _stakeAmounts[i])) {
        highestBidAmount = _bidAmounts[i];
        highestBidder = _bidders[i];
      }
    }
    return (highestBidder, highestBidAmount);
  }

  /*
   * The callback function. Updates the contract state.
   */

  function updateWinner(address _highestBidder, uint _highestBidAmount) public
    {
    winner = _highestBidder;
    winningPrice = _highestBidAmount;
    state = AuctionState.COMPLETED;
    stakeAmounts[_highestBidder] -= winningPrice;
    emit Winner(_highestBidder, _highestBidAmount);
 }

The ``callable`` function 
 This is the function that is computed by a randomly selected 
 SGX node on the Enigma network. In our example,
 this function calculates the highest bidder of the auction. It simply finds 
 the user with the highest corresponding bid and checks whether the bid value 
 is less than or equal to the amount of Ether that the user has staked. The 
 bid is rendered invalid if its value is greater than its prior stake.

The ``callback`` function
 This is the function that is called by the Enigma contract 
 after the ``callable`` has finished. In our auction contract, the ``callback`` will 
 update the state variables ``winner``, ``winningPrice``, and ``state`` as well as 
 decrease the stake of the winner. 


Post-Auction: Withdrawls
^^^^^^^^^^^^^^^^^^^^^^^^^
Bidders who did not win the auction withdawl all their prior staked ether, and the winner 
can claim their reward. ::

 function withdraw() external {
   require(state == AuctionState.COMPLETED);
   require(stakeAmounts[msg.sender] > 0);
   uint amount = stakeAmounts[msg.sender];
   stakeAmounts[msg.sender] = 0;
   msg.sender.transfer(amount);
 }

The winners rewards are claimed by calling the ``claimReward`` function, which mints 
an ERC721 token specified in the ``Auction Factory``. The winner does not need to send 
any Ether, as the contract takes the bid amount from their stake. ::

 function claimReward() external {
   require(state == AuctionState.COMPLETED);
   require(msg.sender == winner);
   require(!rewardClaimed);
   rewardClaimed = true;
   enigmaCollectible.mintToken(msg.sender, endTime);  // mint an ERC721 Enigma Collectible with arbitrary  tokenID (just use the end time)
 }
 
Finally, the creator of the auction can withdraw the winnerâ€™s stake. ::
 
  function claimEther() external isOwner {
   require(state == AuctionState.COMPLETED);
   msg.sender.transfer(winningPrice);
 }
 
Design Considerations
^^^^^^^^^^^^^^^^^^^^^^
 
Please note that this design has a few limitations: 

**1.** The staking mechanism adds complexity for users as it presents an extra step to the 
bidding process. However, if it is not included in this system, an address could bid an 
extremely high value, become the winner, and not claim the rewardâ€Šâ€”â€Šcausing the auction to 
stay in limbo.

**2.** Since the callable function cannot access contract state in the current release of 
Enigma, the stake amounts of each user is sent as an argument to the callback function. This 
causes the dApp to become an 'oracle' to the Enigma network, since it is responsible for 
retrieving these stake amounts. In a future release, Enigma nodes will be able to call view 
functions of Ethereum contracts in order to address this dependency.

**3.** Auction rewards are minted to simplify the code. In the future, we expect that this 
mechanism will include adding an existing NFT to the auction contract as the item being auctioned.
