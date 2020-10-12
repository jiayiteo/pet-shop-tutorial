Source: https://www.trufflesuite.com/tutorials/pet-shop



This tutorial will take you through the process of building your first dapp---an adoption tracking system for a pet shop!

This tutorial is meant for those with a basic knowledge of Ethereum and smart contracts, who have some knowledge of HTML and JavaScript, but who are new to dapps.

**Note**: For Ethereum basics, please read the Truffle  [Ethereum Overview](https://www.trufflesuite.com/tutorials/ethereum-overview)  tutorial before proceeding.

In this tutorial we will be covering:

1.  Setting up the development environment
2.  Creating a Truffle project using a Truffle Box
3.  Writing the smart contract
4.  Compiling and migrating the smart contract
5.  Testing the smart contract
6.  Creating a user interface to interact with the smart contract
7.  Interacting with the dapp in a browser

## [](https://www.trufflesuite.com/tutorials/pet-shop#background)Background

Pete Scandlon of Pete's Pet Shop is interested in using Ethereum as an efficient way to handle their pet adoptions. The store has space for 16 pets at a given time, and they already have a database of pets. As an initial proof of concept,  **Pete wants to see a dapp which associates an Ethereum address with a pet to be adopted.**

The website structure and styling will be supplied.  **Our job is to write the smart contract and front-end logic for its usage.**

## [](https://www.trufflesuite.com/tutorials/pet-shop#setting-up-the-development-environment)Setting up the development environment

There are a few technical requirements before we start. Please install the following:

-   [Node.js v8+ LTS and npm](https://nodejs.org/en/)  (comes with Node)
-   [Git](https://git-scm.com/)

Once we have those installed, we only need one command to install Truffle:

```shell
npm install -g truffle
```

To verify that Truffle is installed properly, type  `truffle version`  on a terminal. If you see an error, make sure that your npm modules are added to your path.

We also will be using  [Ganache](https://www.trufflesuite.com/ganache), a personal blockchain for Ethereum development you can use to deploy contracts, develop applications, and run tests. You can download Ganache by navigating to  [http://truffleframework.com/ganache](http://truffleframework.com/ganache)  and clicking the "Download" button.

**Note**: If you are developing in an environment without a graphical interface, you can also use Truffle Develop, Truffle's built-in personal blockchain, instead of Ganache. You will need to change some settings---such as the port the blockchain runs on---to adapt the tutorial for Truffle Develop.

## [](https://www.trufflesuite.com/tutorials/pet-shop#creating-a-truffle-project-using-a-truffle-box)Creating a Truffle project using a Truffle Box

1.  Truffle initializes in the current directory, so first create a directory in your development folder of choice and then moving inside it.
    
    ```shell
    mkdir pet-shop-tutorial
    
    cd pet-shop-tutorial
    ```
    
2.  We've created a special  [Truffle Box](https://www.trufflesuite.com/boxes)  just for this tutorial called  `pet-shop`, which includes the basic project structure as well as code for the user interface. Use the  `truffle unbox`  command to unpack this Truffle Box.
    
    ```shell
    truffle unbox pet-shop
    ```
    

**Note**: Truffle can be initialized a few different ways. Another useful initialization command is `truffle init`, which creates an empty Truffle project with no example contracts included. For more information, please see the documentation on  [Creating a project](https://www.trufflesuite.com/docs/truffle/getting-started/creating-a-project).

### [](https://www.trufflesuite.com/tutorials/pet-shop#directory-structure)Directory structure

The default Truffle directory structure contains the following:

-   `contracts/`: Contains the  [Solidity](https://solidity.readthedocs.io/)  source files for our smart contracts. There is an important contract in here called  `Migrations.sol`, which we'll talk about later.
-   `migrations/`: Truffle uses a migration system to handle smart contract deployments. A migration is an additional special smart contract that keeps track of changes.
-   `test/`: Contains both JavaScript and Solidity tests for our smart contracts
-   `truffle-config.js`: Truffle configuration file

The  `pet-shop`  Truffle Box has extra files and folders in it, but we won't worry about those just yet.

## [](https://www.trufflesuite.com/tutorials/pet-shop#writing-the-smart-contract)Writing the smart contract

We'll start our dapp by writing the smart contract that acts as the back-end logic and storage.

1.  Create a new file named  `Adoption.sol`  in the  `contracts/`  directory.
    
2.  Add the following content to the file:
    
    ```solidity
    pragma solidity ^0.5.0;
    
    contract Adoption {
    
    }
    ```
    

Things to notice:

-   The minimum version of Solidity required is noted at the top of the contract:  `pragma solidity ^0.5.0;`. The  `pragma`  command means "_additional information that only the compiler cares about_", while the caret symbol (^) means "_the version indicated or higher_".
-   Like JavaScript or PHP, statements are terminated with semicolons.

### [](https://www.trufflesuite.com/tutorials/pet-shop#variable-setup)Variable setup

Solidity is a statically-typed language, meaning data types like strings, integers, and arrays must be defined.  **Solidity has a unique type called an address**. Addresses are Ethereum addresses, stored as 20 byte values. Every account and smart contract on the Ethereum blockchain has an address and can send and receive Ether to and from this address.

1.  Add the following variable on the next line after  `contract Adoption {`.
    
    ```solidity
    address[16] public adopters;
    ```
    

Things to notice:

-   We've defined a single variable:  `adopters`. This is an  **array**  of Ethereum addresses. Arrays contain one type and can have a fixed or variable length. In this case the type is  `address`  and the length is  `16`.
    
-   You'll also notice  `adopters`  is public.  **Public**  variables have automatic getter methods, but in the case of arrays a key is required and will only return a single value. Later, we'll write a function to return the whole array for use in our UI.
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#your-first-function-adopting-a-pet)Your first function: Adopting a pet

Let's allow users to make adoption requests.

1.  Add the following function to the smart contract after the variable declaration we set up above.
    
    ```solidity
    // Adopting a pet
    function adopt(uint petId) public returns (uint) {
      require(petId >= 0 && petId <= 15);
    
      adopters[petId] = msg.sender;
    
      return petId;
    }
    ```
    

Things to notice:

-   In Solidity the types of both the function parameters and output must be specified. In this case we'll be taking in a  `petId`  (integer) and returning an integer.
    
-   We are checking to make sure  `petId`  is in range of our  `adopters`  array. Arrays in Solidity are indexed from 0, so the ID value will need to be between 0 and 15. We use the  `require()`  statement to ensure the ID is within range.
    
-   If the ID is in range, we then add the address that made the call to our  `adopters`  array.  **The address of the person or smart contract who called this function is denoted by  `msg.sender`**.
    
-   Finally, we return the  `petId`  provided as a confirmation.
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#your-second-function-retrieving-the-adopters)Your second function: Retrieving the adopters

As mentioned above, array getters return only a single value from a given key. Our UI needs to update all pet adoption statuses, but making 16 API calls is not ideal. So our next step is to write a function to return the entire array.

1.  Add the following  `getAdopters()`  function to the smart contract, after the  `adopt()`  function we added above:
    
    ```solidity
    // Retrieving the adopters
    function getAdopters() public view returns (address[16] memory) {
      return adopters;
    }
    ```
    
    Things to notice:
    

-   Since  `adopters`  is already declared, we can simply return it. Be sure to specify the return type (in this case, the type for  `adopters`) as  `address[16] memory`.  `memory`  gives the data location for the variable.
    
-   The  `view`  keyword in the function declaration means that the function will not modify the state of the contract. Further information about the exact limits imposed by view is available  [here](https://solidity.readthedocs.io/en/latest/contracts.html#view-functions).
    

## [](https://www.trufflesuite.com/tutorials/pet-shop#compiling-and-migrating-the-smart-contract)Compiling and migrating the smart contract

Now that we have written our smart contract, the next steps are to compile and migrate it.

### [](https://www.trufflesuite.com/tutorials/pet-shop#compilation)Compilation

Solidity is a compiled language, meaning we need to compile our Solidity to bytecode for the Ethereum Virtual Machine (EVM) to execute. Think of it as translating our human-readable Solidity into something the EVM understands.

1.  In a terminal, make sure you are in the root of the directory that contains the dapp and type:
    
    ```shell
    truffle compile
    ```
    
    **Note**: If you're on Windows and encountering problems running this command, please see the documentation on  [resolving naming conflicts on Windows](https://www.trufflesuite.com/docs/truffle/reference/configuration#resolving-naming-conflicts-on-windows).
    
    You should see output similar to the following:
    
    ```shell
    Compiling your contracts...
    ===========================
    > Compiling ./contracts/Adoption.sol
    > Compiling ./contracts/Migrations.sol
    > Artifacts written to /Users/cruzmolina/Code/truffle-projects/metacoin/build/contracts
    > Compiled successfully using:
       - solc: 0.5.0+commit.1d4f565a.Emscripten.clang
    ```
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#migration)Migration

Now that we've successfully compiled our contracts, it's time to migrate them to the blockchain!

**A migration is a deployment script meant to alter the state of your application's contracts**, moving it from one state to the next. For the first migration, you might just be deploying new code, but over time, other migrations might move data around or replace a contract with a new one.

**Note**: Read more about migrations in the  [Truffle documentation](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations).

You'll see one JavaScript file already in the  `migrations/`  directory:  `1_initial_migration.js`. This handles deploying the  `Migrations.sol`  contract to observe subsequent smart contract migrations, and ensures we don't double-migrate unchanged contracts in the future.

Now we are ready to create our own migration script.

1.  Create a new file named  `2_deploy_contracts.js`  in the  `migrations/`  directory.
    
2.  Add the following content to the  `2_deploy_contracts.js`  file:
    
    ```javascript
    var Adoption = artifacts.require("Adoption");
    
    module.exports = function(deployer) {
      deployer.deploy(Adoption);
    };
    ```
    
3.  Before we can migrate our contract to the blockchain, we need to have a blockchain running. For this tutorial, we're going to use  [Ganache](https://www.trufflesuite.com/ganache), a personal blockchain for Ethereum development you can use to deploy contracts, develop applications, and run tests. If you haven't already,  [download Ganache](https://www.trufflesuite.com/ganache)  and double click the icon to launch the application. This will generate a blockchain running locally on port 7545.
    
    **Note**: Read more about Ganache in the  [Truffle documentation](https://www.trufflesuite.com/docs/ganache/using).
    
    ![Ganache on first launch](https://www.trufflesuite.com/img/tutorials/pet-shop/ganache-initial.png "Ganache on first launch")
    
    Ganache on first launch
    
4.  Back in our terminal, migrate the contract to the blockchain.
    
    ```shell
    truffle migrate
    ```
    
    You should see output similar to the following:
    

```shell
  1_initial_migration.js
  ======================

     Deploying 'Migrations'
     ----------------------
     > transaction hash:    0x3b558e9cdf1231d8ffb3445cb2f9fb01de9d0363e0b97a17f9517da318c2e5af
     > Blocks: 0            Seconds: 0
     > contract address:    0x5ccb4dc04600cffA8a67197d5b644ae71856aEE4
     > account:             0x8d9606F90B6CA5D856A9f0867a82a645e2DfFf37
     > balance:             99.99430184
     > gas used:            284908
     > gas price:           20 gwei
     > value sent:          0 ETH
     > total cost:          0.00569816 ETH


     > Saving migration to chain.
     > Saving artifacts
     -------------------------------------
     > Total cost:          0.00569816 ETH


  2_deploy_contracts.js
  =====================

     Deploying 'Adoption'
     .............................
     .............................
```

You can see the migrations being executed in order, followed by some information related to each migration. (Your information will differ.)

1.  In Ganache, note that the state of the blockchain has changed. The blockchain now shows that the current block, previously  `0`, is now  `4`. In addition, while the first account originally had 100 ether, it is now lower, due to the transaction costs of migration. We'll talk more about transaction costs later.
    
    ![Ganache after migration](https://www.trufflesuite.com/img/tutorials/pet-shop/ganache-migrated.png "Ganache after migration")
    
    Ganache after migration
    

You've now written your first smart contract and deployed it to a locally running blockchain. It's time to interact with our smart contract now to make sure it does what we want.

## [](https://www.trufflesuite.com/tutorials/pet-shop#testing-the-smart-contract-using-solidity)Testing the smart contract using Solidity

Expand This Section

## [](https://www.trufflesuite.com/tutorials/pet-shop#testing-the-smart-contract-using-javascript)Testing the smart contract using JavaScript

Expand This Section

### [](https://www.trufflesuite.com/tutorials/pet-shop#running-the-tests)Running the tests

1.  Back in the terminal, run the tests:
    
    ```shell
    truffle test
    ```
    
2.  If all the tests pass, you'll see console output similar to this:
    

```shell
   Using network 'development'.

   Compiling your contracts...
   ===========================
   > Compiling ./test/TestAdoption.sol
   > Artifacts written to /var/folders/z3/v0sd04ys11q2sh8tq38mz30c0000gn/T/test-11934-19747-g49sra.0ncrr
   > Compiled successfully using:
      - solc: 0.5.0+commit.1d4f565a.Emscripten.clang

     TestAdoption
       ✓ testUserCanAdoptPet (91ms)
       ✓ testGetAdopterAddressByPetId (70ms)
       ✓ testGetAdopterAddressByPetIdInArray (89ms)


     3 passing (670ms)
```

## [](https://www.trufflesuite.com/tutorials/pet-shop#creating-a-user-interface-to-interact-with-the-smart-contract)Creating a user interface to interact with the smart contract

Now that we've created the smart contract, deployed it to our local test blockchain and confirmed we can interact with it via the console, it's time to create a UI so that Pete has something to use for his pet shop!

Included with the  `pet-shop`  Truffle Box was code for the app's front-end. That code exists within the  `src/`  directory.

The front-end doesn't use a build system (webpack, grunt, etc.) to be as easy as possible to get started. The structure of the app is already there; we'll be filling in the functions which are unique to Ethereum. This way, you can take this knowledge and apply it to your own front-end development.

### [](https://www.trufflesuite.com/tutorials/pet-shop#instantiating-web3)Instantiating web3

1.  Open  `/src/js/app.js`  in a text editor.
    
2.  Examine the file. Note that there is a global  `App`  object to manage our application, load in the pet data in  `init()`  and then call the function  `initWeb3()`. The  [web3 JavaScript library](https://github.com/ethereum/web3.js/)  interacts with the Ethereum blockchain. It can retrieve user accounts, send transactions, interact with smart contracts, and more.
    
3.  Remove the multi-line comment from within  `initWeb3`  and replace it with the following:
    
    ```javascript
    // Modern dapp browsers...
    if (window.ethereum) {
      App.web3Provider = window.ethereum;
      try {
        // Request account access
        await window.ethereum.enable();
      } catch (error) {
        // User denied account access...
        console.error("User denied account access")
      }
    }
    // Legacy dapp browsers...
    else if (window.web3) {
      App.web3Provider = window.web3.currentProvider;
    }
    // If no injected web3 instance is detected, fall back to Ganache
    else {
      App.web3Provider = new Web3.providers.HttpProvider('http://localhost:7545');
    }
    web3 = new Web3(App.web3Provider);
    ```
    

Things to notice:

-   First, we check if we are using modern dapp browsers or the more recent versions of  [MetaMask](https://github.com/MetaMask)  where an  `ethereum`  provider is injected into the  `window`  object. If so, we use it to create our web3 object, but we also need to explicitly request access to the accounts with  `ethereum.enable()`.
    
-   If the  `ethereum`  object does not exist, we then check for an injected  `web3`  instance. If it exists, this indicates that we are using an older dapp browser (like  [Mist](https://github.com/ethereum/mist)  or an older version of MetaMask). If so, we get its provider and use it to create our web3 object.
    
-   If no injected web3 instance is present, we create our web3 object based on our local provider. (This fallback is fine for development environments, but insecure and not suitable for production.)
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#instantiating-the-contract)Instantiating the contract

Now that we can interact with Ethereum via web3, we need to instantiate our smart contract so web3 knows where to find it and how it works. Truffle has a library to help with this called  `@truffle/contract`. It keeps information about the contract in sync with migrations, so you don't need to change the contract's deployed address manually.

1.  Still in  `/src/js/app.js`, remove the multi-line comment from within  `initContract`  and replace it with the following:
    
    ```javascript
    $.getJSON('Adoption.json', function(data) {
      // Get the necessary contract artifact file and instantiate it with @truffle/contract
      var AdoptionArtifact = data;
      App.contracts.Adoption = TruffleContract(AdoptionArtifact);
    
      // Set the provider for our contract
      App.contracts.Adoption.setProvider(App.web3Provider);
    
      // Use our contract to retrieve and mark the adopted pets
      return App.markAdopted();
    });
    ```
    

Things to notice:

-   We first retrieve the artifact file for our smart contract.  **Artifacts are information about our contract such as its deployed address and Application Binary Interface (ABI)**.  **The ABI is a JavaScript object defining how to interact with the contract including its variables, functions and their parameters.**
    
-   Once we have the artifacts in our callback, we pass them to  `TruffleContract()`. This creates an instance of the contract we can interact with.
    
-   With our contract instantiated, we set its web3 provider using the  `App.web3Provider`  value we stored earlier when setting up web3.
    
-   We then call the app's  `markAdopted()`  function in case any pets are already adopted from a previous visit. We've encapsulated this in a separate function since we'll need to update the UI any time we make a change to the smart contract's data.
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#getting-the-adopted-pets-and-updating-the-ui)Getting The Adopted Pets and Updating The UI

1.  Still in  `/src/js/app.js`, remove the multi-line comment from  `markAdopted`  and replace it with the following:
    
    ```javascript
    var adoptionInstance;
    
    App.contracts.Adoption.deployed().then(function(instance) {
      adoptionInstance = instance;
    
      return adoptionInstance.getAdopters.call();
    }).then(function(adopters) {
      for (i = 0; i < adopters.length; i++) {
        if (adopters[i] !== '0x0000000000000000000000000000000000000000') {
          $('.panel-pet').eq(i).find('button').text('Success').attr('disabled', true);
        }
      }
    }).catch(function(err) {
      console.log(err.message);
    });
    ```
    

Things to notice:

-   We access the deployed  `Adoption`  contract, then call  `getAdopters()`  on that instance.
    
-   We first declare the variable  `adoptionInstance`  outside of the smart contract calls so we can access the instance after initially retrieving it.
    
-   Using  **call()**  allows us to read data from the blockchain without having to send a full transaction, meaning we won't have to spend any ether.
    
-   After calling  `getAdopters()`, we then loop through all of them, checking to see if an address is stored for each pet. Since the array contains address types, Ethereum initializes the array with 16 empty addresses. This is why we check for an empty address string rather than null or other falsey value.
    
-   Once a  `petId`  with a corresponding address is found, we disable its adopt button and change the button text to "Success", so the user gets some feedback.
    
-   Any errors are logged to the console.
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#handling-the-adopt-function)Handling the adopt() Function

1.  Still in  `/src/js/app.js`, remove the multi-line comment from  `handleAdopt`  and replace it with the following:
    
    ```javascript
    var adoptionInstance;
    
    web3.eth.getAccounts(function(error, accounts) {
      if (error) {
        console.log(error);
      }
    
      var account = accounts[0];
    
      App.contracts.Adoption.deployed().then(function(instance) {
        adoptionInstance = instance;
    
        // Execute adopt as a transaction by sending account
        return adoptionInstance.adopt(petId, {from: account});
      }).then(function(result) {
        return App.markAdopted();
      }).catch(function(err) {
        console.log(err.message);
      });
    });
    ```
    

Things to notice:

-   We use web3 to get the user's accounts. In the callback after an error check, we then select the first account.
    
-   From there, we get the deployed contract as we did above and store the instance in  `adoptionInstance`. This time though, we're going to send a  **transaction**  instead of a call. Transactions require a "from" address and have an associated cost. This cost, paid in ether, is called  **gas**. The gas cost is the fee for performing computation and/or storing data in a smart contract. We send the transaction by executing the  `adopt()`  function with both the pet's ID and an object containing the account address, which we stored earlier in  `account`.
    
-   The result of sending a transaction is the transaction object. If there are no errors, we proceed to call our  `markAdopted()`  function to sync the UI with our newly stored data.
    

## [](https://www.trufflesuite.com/tutorials/pet-shop#interacting-with-the-dapp-in-a-browser)Interacting with the dapp in a browser

Now we're ready to use our dapp!

### [](https://www.trufflesuite.com/tutorials/pet-shop#installing-and-configuring-metamask)Installing and configuring MetaMask

The easiest way to interact with our dapp in a browser is through  [MetaMask](https://metamask.io/), a browser extension for both Chrome and Firefox.

1.  Install MetaMask in your browser.
    
2.  Once installed, a tab in your browser should open displaying the following:
    
    ![Welcome to MetaMask](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-welcome.png "Welcome to MetaMask")
    
    Welcome to MetaMask
    
3.  After clicking  **Getting Started**, you should see the initial MetaMask screen. Click  **Import Wallet**.
    
    ![Initial screen](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-initial.png "Initial screen")
    
    MetaMask initial screen
    
4.  Next, you should see a screen requesting anonymous analytics. Choose to decline or agree.
    
    ![Improve MetaMask](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-analytics.png "Improve MetaMask")
    
    Improve MetaMask
    
5.  In the box marked  **Wallet Seed**, enter the mnemonic that is displayed in Ganache.
    
    **Warning**: Do not use this mnemonic on the main Ethereum network (mainnet). If you send ETH to any account generated from this mnemonic, you will lose it all!
    
    Enter a password below that and click  **OK**.
    
    ![MetaMask seed phrase](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-seed.png "MetaMask seed phrase")
    
    MetaMask seed phrase
    
6.  If all goes well, MetaMask should display the following screen. Click  **All Done**.
    
    ![Congratulations](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-congratulations.png "Congratulations")
    
    Congratulations
    
7.  Now we need to connect MetaMask to the blockchain created by Ganache. Click the menu that shows "Main Network" and select  **Custom RPC**.
    
    ![MetaMask network menu](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-networkmenu.png "MetaMask network menu")
    
    MetaMask network menu
    
8.  In the box titled "New Network" enter  `http://127.0.0.1:7545`  and click  **Save**.
    
    ![MetaMask Custom RPC](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-customrpc.png "MetaMask Custom RPC")
    
    MetaMask Custom RPC
    
    The network name at the top will switch to say  `http://127.0.0.1:7545`.
    
9.  Click the top-right X to close out of Settings and return to the Accounts page.
    
    Each account created by Ganache is given 100 ether. You'll notice it's slightly less on the first account because some gas was used when the contract itself was deployed and when the tests were run.
    
    ![MetaMask account configured](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-account1.png "MetaMask account configured")
    
    MetaMask account configured
    
    Configuration is now complete.
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#installing-and-configuring-lite-server)Installing and configuring lite-server

We can now start a local web server and use the dapp. We're using the  `lite-server`  library to serve our static files. This shipped with the  `pet-shop`  Truffle Box, but let's take a look at how it works.

1.  Open  `bs-config.json`  in a text editor (in the project's root directory) and examine the contents:
    
    ```javascript
    {
      "server": {
        "baseDir": ["./src", "./build/contracts"]
      }
    }
    ```
    
    This tells  `lite-server`  which files to include in our base directory. We add the  `./src`  directory for our website files and  `./build/contracts`  directory for the contract artifacts.
    
    We've also added a  `dev`  command to the  `scripts`  object in the  `package.json`  file in the project's root directory. The  `scripts`  object allows us to alias console commands to a single npm command. In this case we're just doing a single command, but it's possible to have more complex configurations. Here's what yours should look like:
    
    ```javascript
    "scripts": {
      "dev": "lite-server",
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    ```
    
    This tells npm to run our local install of  `lite-server`  when we execute  `npm run dev`  from the console.
    

### [](https://www.trufflesuite.com/tutorials/pet-shop#using-the-dapp)Using the dapp

1.  Start the local web server:
    
    ```shell
    npm run dev
    ```
    
    The dev server will launch and automatically open a new browser tab containing your dapp.
    
    ![Pete's Pet Shop](https://www.trufflesuite.com/img/tutorials/pet-shop/dapp.png "Pete's Pet Shop")
    
    Pete's Pet Shop
    
2.  A MetaMask pop-up should appear requesting your approval to allow Pete's Pet Shop to connect to your MetaMask wallet. Without explicit approval, you will be unable to interact with the dapp. Click  **Connect**.
    
    ![MetaMask approval request](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-transactionconfirm.png "MetaMask approval request")
    
    MetaMask approval request
    
3.  To use the dapp, click the  **Adopt**  button on the pet of your choice.
    
4.  You'll be automatically prompted to approve the transaction by MetaMask. Click  **Submit**  to approve the transaction.
    
    ![Adoption transaction review](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-transactionconfirm.png "Adoption transaction review")
    
    Adoption transaction review
    
5.  You'll see the button next to the adopted pet change to say "Success" and become disabled, just as we specified, because the pet has now been adopted.
    
    ![Adoption success](https://www.trufflesuite.com/img/tutorials/pet-shop/dapp-success.png "Adoption success")
    
    Adoption success
    
    **Note**: If the button doesn't automatically change to say "Success", refreshing the app in the browser should trigger it.
    
    And in MetaMask, you'll see the transaction listed:
    
    ![MetaMask transaction](https://www.trufflesuite.com/img/tutorials/pet-shop/metamask-transactionsuccess.png "MetaMask transaction")
    
    MetaMask transaction
    
    You'll also see the same transaction listed in Ganache under the "Transactions" section.
    

Congratulations! You have taken a huge step to becoming a full-fledged dapp developer. For developing locally, you have all the tools you need to start making more advanced dapps. If you'd like to make your dapp live for others to use, stay tuned for our future tutorial on deploying to the Ropsten testnet.
