Creating a React Front-End
===========================
This section is dedicated to creating a simple graphical front-end for the 
Millionares Problem example using React, a JavaScript library for building
component-oriented user interfaces. The ``enigma-template-dapp`` repository is 
based off of `truffle-react box variant <https://github.com/adrianmcli/truffle-react>`__,
which allows for the rapid deployment of a unique front-end with the related packages
and logic separated from the code - in this case, located in the ``/client/src`` folder. 

Please note that the below section is almost entirely generic React code that does not
have any role in interfacing with the secret contracts made earlier. There are a couple
small unique portions, Noted by their comment annotations within the code sections below.

If you are unfamiliar with React, it is advised to read `this quick tutorial 
<https://reactjs.org/tutorial/tutorial.html>`__ to get a grasp of the basic concepts and
components involved.

The full source code for the demo application that is built using this section can be found
`here <https://github.com/enigmampc/enigma-template-dapp/tree/millionaires_problem_demo/client/src>`__.

**1.** Enter the ``/client/src`` folderof the ``enigma-template-dapp`` directory.

**2.** Open up ``App.js`` in your favorite text editor and paste the following code: ::

 import React, { Component } from "react";
 import getContractInstance from "./utils/getContractInstance";
 import EnigmaSetup from "./utils/getEnigmaSetup";
 import millionairesProblemFactoryContractDefinition from "./contracts/MillionairesProblemFactory.json";
 import millionairesProblemContractDefinition from "./contracts/MillionairesProblem.json";
 import { Container, Message } from "semantic-ui-react";
 import Header from "./Header";
 import MillionairesProblemWrapper from "./MillionairesProblemWrapper";
 import Paper from "@material-ui/core/Paper";
 import "./App.css";
 const GAS = "1000000";

 class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      enigmaSetup: null,
      millionairesProblemFactory: null,
      millionairesProblem: null
    };
  }

  componentDidMount = async () => {
    /*
    Initialize bundled object containing web3, accounts, Enigma/EnigmaToken contracts, and 
    enigma/principal wrappers
    */
    let enigmaSetup = new EnigmaSetup();
    await enigmaSetup.init();
    // Initialize MillionairesProblemFactory contract instance
    const millionairesProblemFactory = await getContractInstance(
      enigmaSetup.web3,
      millionairesProblemFactoryContractDefinition
    );
    const millionairesProblems = await millionairesProblemFactory.getMillionairesProblems.call();
    // If we've deployed at least one MillionaireProblem, get the latest one
    if (millionairesProblems.length != 0) {
      const millionairesProblem = await getContractInstance(
        enigmaSetup.web3,
        millionairesProblemContractDefinition,
        millionairesProblems[millionairesProblems.length - 1]
      );
      this.setState({
        millionairesProblem
      });
    }

    this.setState({ enigmaSetup, millionairesProblemFactory });
  };

  // Create fresh, new MillionaireProblem contract
  async createNewMillionairesProblem() {
    await this.state.millionairesProblemFactory.createNewMillionairesProblem({
      from: this.state.enigmaSetup.accounts[0],
      gas: GAS
    });

    // Obtain the latest-deployed MillionaireProblem instance, this will be the one we interact with
    const millionairesProblems = await this.state.millionairesProblemFactory.getMillionairesProblems.call();
    const millionairesProblem = await getContractInstance(
      this.state.enigmaSetup.web3,
      millionairesProblemContractDefinition,
      millionairesProblems[millionairesProblems.length - 1]
    );
    this.setState({
      millionairesProblem
    });
  }

  render() {
    if (!this.state.enigmaSetup) {
      return (
        <div className="App">
          <Header />
          <Message color="red">Enigma setup still loading...</Message>
        </div>
      );
    } else {
      return (
        <div className="App">
          <Header />
          <br />
          <Container>
            <Paper>
              <MillionairesProblemWrapper
                onCreateNewMillionaresProblem={() => {
                  this.createNewMillionairesProblem();
                }}
                enigmaSetup={this.state.enigmaSetup}
                millionairesProblem={this.state.millionairesProblem}
              />
            </Paper>
          </Container>
        </div>
      );
    }
  }
 }
 export default App; 

The file ``App.js`` is essentially the index page of your dApps interface, which will handle:

* Initialization of all the ``EnigmaSetup`` object [lines 28–29] mentioned above and the top-level rendering of the dApp.
    
* Pulling in the contract ABIs for our custom contracts [lines 4–5] and appropriately setting the state for the current ``MillionairesProblem`` if it exists [lines 35–48].

**3.** In the same directory, create a file named ``MillionairesProblemWrapper.js`` and paste the following code: ::

 import React, { Component } from "react";
 import PropTypes from "prop-types";
 import { withStyles } from "@material-ui/core/styles";
 import Button from "@material-ui/core/Button";
 import { Message } from "semantic-ui-react";
 import AddMillionaireDialog from "./AddMillionaireDialog";
 const engUtils = require("./lib/enigma-utils");
 // Specify the signature for the callable and callback functions, make sure there are NO spaces
 const CALLABLE = "computeRichest(address[],uint[])";
 const CALLBACK = "setRichestAddress(address)";
 const ENG_FEE = 1; 
 const GAS = "1000000";

 const styles = theme => ({
	button: {
		display: "block",
		marginTop: theme.spacing.unit * 2
	}
 });

 class MillionairesProblemWrapper extends Component {
	constructor(props) {
		super(props);
		this.state = {
			numMillionaires: null,
			richestAddress: "TBD"
		};
		this.handleSubmit = this.handleSubmit.bind(this);
		this.addMillionaire = this.addMillionaire.bind(this);
	}

	componentDidMount = async () => {
		/*
		Check if we have an instance of the MillionairesProblem deployed or not before
		we call any functions on it
		*/
		if (this.props.millionairesProblem != null) {
			let numMillionaires = await this.props.millionairesProblem.numMillionaires.call();
			numMillionaires = numMillionaires.toNumber();
			this.setState({ numMillionaires });
		}
	};

	// Handles re-rendering if we've created a new MillionairesProblem (callback resides in parent)
	async componentWillReceiveProps(nextProps) {
		if (this.props.millionairesProblem != nextProps.millionairesProblem) {
			this.setState({ numMillionaires: 0, richestAddress: "TBD" });
		}
	}

	/*
	Callback for adding a new millionaire. Note that we are encrypting data 
	(address and net worth) in this function and pass in those values to the contract
	*/
	async addMillionaire(address, netWorth) {
		let encryptedAddress = getEncryptedValue(address);
		let encryptedNetWorth = getEncryptedValue(netWorth);
		await this.props.millionairesProblem.addMillionaire(
			encryptedAddress,
			encryptedNetWorth,
			{ from: this.props.enigmaSetup.accounts[0], gas: GAS }
		);
		let numMillionaires = await this.props.millionairesProblem.numMillionaires.call();
		numMillionaires = numMillionaires.toNumber();
		this.setState({ numMillionaires });
	}

	/*
	Creates an Enigma task to be computed by the network.
	*/
	async enigmaTask() {
		let numMillionaires = await this.props.millionairesProblem.numMillionaires.call();
		let encryptedAddresses = [];
		let encryptedNetWorths = [];
		// Loop through each millionaire to construct a list of encrypted addresses and net worths
		for (let i = 0; i < numMillionaires; i++) {
			// Obtain the encrypted address and net worth for a particular millionaire
			let encryptedValue = await this.props.millionairesProblem.getInfoForMillionaire.call(
				i
			);
			encryptedAddresses.push(encryptedValue[0]);
			encryptedNetWorths.push(encryptedValue[1]);
		}
		let blockNumber = await this.props.enigmaSetup.web3.eth.getBlockNumber();
		/*
		Take special note of the arguments passed in here (blockNumber, dappContractAddress, 
		callable, callableArgs, callback, fee, preprocessors). This is the critical step for how
		you run the secure computation from your front-end!!!
		*/
		let task = await this.props.enigmaSetup.enigma.createTask(
			blockNumber,
			this.props.millionairesProblem.address,
			CALLABLE,
			[encryptedAddresses, encryptedNetWorths],
			CALLBACK,
			ENG_FEE,
			[]
		);
		let resultFee = await task.approveFee({
			from: this.props.enigmaSetup.accounts[0],
			gas: GAS
		});
		let result = await task.compute({
			from: this.props.enigmaSetup.accounts[0],
			gas: GAS
		});
		console.log("got tx:", result.tx, "for task:", task.taskId, "");
		console.log("mined on block:", result.receipt.blockNumber);
	}

	// onClick listener for Check Richest button, will call the enigmaTask from here
	async handleSubmit(event) {
		event.preventDefault();
		let richestAddress = "Computing richest...";
		this.setState({ richestAddress });
		// Run the enigma task secure computation above
		await this.enigmaTask();
		// Watch for event and update state once callback is completed/event emitted
		const callbackFinishedEvent = this.props.millionairesProblem.CallbackFinished();
		callbackFinishedEvent.watch(async (error, result) => {
			richestAddress = await this.props.millionairesProblem.richestMillionaire.call();
			this.setState({ richestAddress });
		});
	}

	render() {
		const { classes } = this.props;
		if (this.state.numMillionaires == null) {
			return (
				<div>
					<Button onClick={this.props.onCreateNewMillionaresProblem}>
						{"Create New Millionaires' Problem"}
					</Button>
				</div>
			);
		} else {
			return (
				<div>
					<Button
						onClick={this.props.onCreateNewMillionaresProblem}
						variant="contained"
					>
						{"Create New Millionaires' Problem"}
					</Button>
					<h2>Num Millionaires = {this.state.numMillionaires}</h2>
					<h2>Richest Millionaire = {this.state.richestAddress}</h2>
					<AddMillionaireDialog
						accounts={this.props.enigmaSetup.accounts}
						onAddMillionaire={this.addMillionaire}
					/>
					<br />
					<Button
						onClick={this.handleSubmit}
						disabled={this.state.numMillionaires == 0}
						variant="contained"
						color="secondary"
					>
						Check Richest
					</Button>
				</div>
			);
		}
	}
 }

 // Function to encrypt values (in this case either address or net worth)
 function getEncryptedValue(value) {
	let clientPrivKey =
		"853ee410aa4e7840ca8948b8a2f67e9a1c2f4988ff5f4ec7794edf57be421ae5";
	let enclavePubKey =
		"0061d93b5412c0c99c3c7867db13c4e13e51292bd52565d002ecf845bb0cfd8adfa5459173364ea8aff3fe24054cca88581f6c3c5e928097b9d4d47fce12ae47";
	let derivedKey = engUtils.getDerivedKey(enclavePubKey, clientPrivKey);
	let encrypted = engUtils.encryptMessage(derivedKey, value);

	return encrypted;
 }

 MillionairesProblemWrapper.propTypes = {
	classes: PropTypes.object.isRequired
 };

 export default withStyles(styles)(MillionairesProblemWrapper);

The ``MillionairesProblemWrapper.js`` component drives the business logic for the MillionairesProblem that’s being rendered and handles:

* The logic to ``addMillionaire`` (committing encrypted values for the address and net worth to the contract) [lines 51–66].
* Rendering the current state of the contract (number of participants, richest address) [lines 145–146].
* The ``enigmaTask`` (in charge of executing the secret computation, running the callable and callback functions with the encrypted arguments necessary) [lines 68–109].


**4.** In the same directory, create a file named ``AddMillionaireDialog.js`` and paste the following code: ::

 import React, { Component } from "react";
 import PropTypes from "prop-types";
 import { withStyles } from "@material-ui/core/styles";
 import Button from "@material-ui/core/Button";
 import Dialog from "@material-ui/core/Dialog";
 import DialogActions from "@material-ui/core/DialogActions";
 import DialogContent from "@material-ui/core/DialogContent";
 import DialogContentText from "@material-ui/core/DialogContentText";
 import DialogTitle from "@material-ui/core/DialogTitle";
 import Input from "@material-ui/core/Input";
 import InputLabel from "@material-ui/core/InputLabel";
 import FormControl from "@material-ui/core/FormControl";
 import MenuItem from "@material-ui/core/MenuItem";
 import Select from "@material-ui/core/Select"; 

 const styles = theme => ({
  button: {
    display: "block",
    marginTop: theme.spacing.unit * 2
  },
  formControl: {
    margin: theme.spacing.unit,
    minWidth: 120
  }
 });

 class AddMillionaireDialog extends Component {
  constructor(props) {
    super(props);
    this.state = {
      open: false,
      millionaireAddress: "None",
      millionaireNetWorth: null
    };
    this.handleChangeAddress = this.handleChangeAddress.bind(this);
    this.handleChangeNetWorth = this.handleChangeNetWorth.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleClickOpen = () => {
    this.setState({ open: true });
  };

  handleClose = () => {
    this.setState({ open: false });
  };

  // onChange listener to update state with user-input address
  handleChangeAddress(event) {
    this.setState({ millionaireAddress: event.target.value });
  }

  // onChange listener to update state with user-input net worth
  handleChangeNetWorth(event) {
    this.setState({ millionaireNetWorth: event.target.value });
  }

  // onClick listener to update to trigger addMillionaire callback from parent component
  async handleSubmit(event) {
    event.preventDefault();
    // Trigger MillionairesProblemWrapper addMillionaire callback
    this.props.onAddMillionaire(
      this.state.millionaireAddress,
      this.state.millionaireNetWorth
    );
    this.setState({
      open: false,
      millionaireAddress: "None",
      millionaireNetWorth: null
    });
  }

  render() {
    const { classes } = this.props;
    return (
      <div>
        <Button
          onClick={this.handleClickOpen}
          variant="contained"
          color="primary"
        >
          Add Millionaire
        </Button>
        <Dialog
          open={this.state.open}
          onClose={this.handleClose}
          aria-labelledby="form-dialog-title"
        >
          <DialogTitle id="form-dialog-title">Add Millionaire</DialogTitle>
          <DialogContent>
            <DialogContentText>
              To add yourself, please set your address and state your net
              worth...
            </DialogContentText>
            <form className={classes.root} onSubmit={this.handleSubmit}>
              <FormControl className={classes.formControl}>
                <InputLabel htmlFor="millionaireAddress">
                  Millionaire Address
                </InputLabel>
                <Select
                  value={this.state.millionaireAddress}
                  onChange={this.handleChangeAddress}
                  inputProps={{
                    name: "address",
                    id: "millionaireAddress"
                  }}
                >
                  <MenuItem value="">
                    <em>None</em>
                  </MenuItem>
                  {this.props.accounts.map((account, i) => {
                    return (
                      <MenuItem key={i} value={account}>
                        {account}
                      </MenuItem>
                    );
                  })}
                </Select>
              </FormControl>
              <FormControl className={classes.formControl}>
                <InputLabel htmlFor="millionaireNetWorth">
                  Millionaire Net Worth
                </InputLabel>
                <Input
                  id="millionaireNetWorth"
                  onChange={this.handleChangeNetWorth}
                  autoComplete="off"
                />
              </FormControl>
            </form>
          </DialogContent>
          <DialogActions>
            <Button onClick={this.handleClose} color="primary">
              Cancel
            </Button>
            <Button onClick={this.handleSubmit}>Add Millionaire</Button>
          </DialogActions>
        </Dialog>
      </div>
    );
  }
 }

 AddMillionaireDialog.propTypes = {
  classes: PropTypes.object.isRequired
 };

 export default withStyles(styles)(AddMillionaireDialog);

This ``AddMillionaireDialog.js`` file is a `material-ui <https://material-ui.com>`__-driven dialog script which handles:

* Inputting address and net worth [lines 48–56].
* Triggering the ``addMillionaire`` callback from the above file, which commits the encrypted address and net worth to the active ``MillionaireProblem`` contract [lines 58–71].

**5.** To build your changes, simply rereun ``npm run start`` in the ``/client`` folder. You should now see your new interface loaded. 

Conclusion
~~~~~~~~~~~
That's it! By now you have successfully deployed the Enigma Docker Test Network, 
created a template dApp and built a graphical front-end using React. For more 
information on how Enigma functions, refer to the following few sections.

**Have questions or issues?** Stop by our `developer forum <https://forum.enigma.co>`__ and let us know!

