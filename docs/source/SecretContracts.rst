
Writing a Secret Contract
=========================

Secret contracts are smart contracts that execute code over Enigma's decentralized 
network without revealing input data, ensuring both integrity and confidentiality. Secret 
contracts are written in a programming language called Solidity, and are implemented 
very similarly to Ethereum-based smart contracts.

**NOTICE:** Although the ``enigma-docker-network`` must stay running, at this point in the 
guide you will be operating entirely out of the 
``enigma-template-dapp`` folder (the ``dapp`` terminal). 

Example: The Millionaires Problem
``````````````````````````````````

The millionaires problem is a simple concept with complex solutions: How do two 
individuals compare their net worth without ever exposing their actual value to 
the other party?

Inside your ``dapp`` terminal, enter the ``/contracts`` folder and create a new 
file called ``MillionairesProbblem.sol`` and paste the following contract code within
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
  This is a public function that runs secret computations inside the SGX enclave. It is a   ``pure``
  function, meaning it does not read from nor write to the contract slate, but computes solely
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
  altering the contract state. In this example, we input the ``_address`` we’ve obtained from the 
  ``callable``, store it as the ``richestMillionaire`` state variable, and emit the 
  ``CallbackFinished`` event. The output of the final event is important, as it is possible to set
  up and event watcher within the front-end to perform a task upon successful completion.

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

**NOTICE:** To create new contracts or modify existing ones, you must redeploy to the network with the 
following command: ``darq-truffle migrate --reset --network development``
