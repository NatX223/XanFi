# XanFi
Cross-chain asset manager Dapp. Powered by Wormhole.

live link - 

> ## Table of Contents

-   [Problem Statement](#Problem-statement)
-   [Solution](#Solution)
-   [How it works](#How-it-works)
-   [Technologies Used](#technologies-used)
    -   [Smart Contract](#Solidity-smart-contracts)
    -   [Wormhole](#Wormhole)
    -   [Firebase](#Firebase)
    -   [Next.js](#Next.js)
-   [Deployment](#Deployment)
#

> ## Problem-statement

Problem Statement: In the rapidly growing world of decentralized finance (DeFi), investors struggle to efficiently diversify their portfolios across multiple blockchains. Traditional index funds are restricted to single chains and lack the ability to manage cross-chain assets seamlessly. Additionally, asset management for multi-chain portfolios involves complex processes like manual swaps, cross-chain transfers, and the inefficiency of tracking prices and balances across disparate protocols.

> ## Solution

Solution: The proposed project solves this problem by introducing a decentralized, cross-chain index fund protocol that allows users to create, invest in, and manage multi-chain index funds. The solution leverages the Wormhole protocol for cross-chain messaging, enabling the deployment and management of index fund contracts across multiple blockchains. The protocol offers automated price calculations, seamless investing and redeeming mechanisms, and flexible asset replacement features. With decentralized exchange (DEX) integrations, the solution handles asset swaps and cross-chain transfers autonomously, making diversified investment in DeFi simpler and more efficient.

> ## How-it-works

    -   **Index Creation** An Index is created by filling out the details of the index such as the name, description, category 
    and assets that will make up the index. The creation process starts with the factory contract as it deploys the index contract 
    on all supported chains, this is powered by wormhole messaging.
    -   **Calculations** 
        - Price Calculations - The price for each is index is calculated just as in tradFi i.e
                total value of underlying assets
        price = --------------------------------
                 total supply of tokens(shares)
        search how to include calculations and Symbols
        add total value of underlying assets calculation
        - Asset Sell Amount - To get the amount of an underlying asset to be sold off when a user sells of their index tokens, the
        below formula is used
                            amount of index tokens * amount of asset tokens held by index smart contract
        token sell amount = ----------------------------------------------------------------------------
                                                    Total supply of index tokens
    -   **Investing** To invest in an index i.e to buy the index tokens, a user just needs to specify the amount of USDT they want 
    to use in the purchase in the index page and sign the transaction that pops up. When the transaction is signed, the specified 
    amount of USDT is sent to the index contract, which is then used to buy the corresponding amount the constituent assets. Index 
    tokens are then minted to the investor.
    -   **Selling** To sell off index tokens a user enters the index page and specifies the amount of tokens they want to sell off.
    On the smart contracts the corresponding amount of each constituent asset is sold off in exchange for USDT and the proceeds is 
    sent to the investor's specified address and the amount of tokens specified is burned from the investor's address.
    -   **Asset Replacement** In the event that an asset in an index does not perform well or for what ever reason an index creator 
    can choose to replace an asset in the index, the asset being replaced and the asset replacing it will be specified. When an asset
    is been replaced the tokens of the replaced asset in the index smart contract are sold off and the proceeds from that sale are then
    used to purchase the new asset add to the index.

> ## Technologies Used

| <b><u>Stack</u></b>      | <b><u>UsageSummary</u></b>                           |
| :----------------------- | :--------------------------------------------------- |
| **`Solidity`**           | Smart contract                                       |
| **`Wormhole`**           | Cross-chain messaging and token transfers            |
| **`Firebase`**           | Off-chain records                                    |
| **`React.js & Next.js`** | Frontend                                             |

-   ### **Solidity smart contracts**

    The  smart contracts can be found [here]()

    -   **Index Factory Contract** The Index Factory contract as the name suggests, is used in deploying and initializing index fund contracts. This contract is a protocol contract and as such it is deployed and managed by the protocol. The factory contract is deployed on multiple supported chains, this is done to enable interoperability between the supported chains by sending cross-chain messages to factory contracts on the supported chains when an index fund contract is deployed. The IndexFactory contract code can be found [here]().

    -   **Router Contract** The Router contract just like the factory contract is a protocol contract and is deployed on the various supported chains. The 
    main purpose of the router contract is to enable the selling off of an index fund's assets held on another chain. When an investor wants to liquidate 
    their fund tokens, the crossChainRedeem function is called on the router contract and a message is sent to the target router contract, which then calls 
    a function on the corresponding index fund contract on the target chain that sells off the appropriate amount of assets and sends the the gotten USDT 
    to the investor who called the function.  
    The Router contract code can be found [here]().

    -   **Index Fund Contract** The Index Contract is the main contract of the protocol. Users can deploy their own index fund contract from the factory 
    contract. When an index fund contract is deployed it is also initialized with the various needed parameters such as the asset contracts and the their 
    ratios. This contract handles investing in a fund(buying shares) and selling off shares. Investing in the fund is enabled by the invest function wihich 
    takes in the amount in USDT(the purchase token) the user wants to invest in the fund and the index contracts on other chains, This function calculates 
    the amount to be used in purchasing the various index assets and calls a DEX swap function to purchase these assets, for cross-chains assets it sends a 
    payload with tokens to the assets chain index contract to excute the purchase. The corresponding amount of index fund tokens are then minted to the 
    caller of the function. The redeem function handles the selling off of the index fund shares tokens, it calculates the amount of the assets tokens it 
    needs to sell off and calls DEX swap function to sell the assets and sends the USDT gotten from the swap to the caller of the function and then the 
    amount of index fund shares tokens are burned from the callers balance. Other functions in this contract include the sale function which is called by 
    the Router contract to execute cross-chain sell off functions, the replaceAsset function that replaces an asset in the index with another - it does 
    this by selling off all tokens of the replaced asset and buying tokens of the new asset with the USDT gotten from the sell off of the replaced asset. 
    The code for the index contract can be found [here](). 

    <!-- how to deploy -->
    <!-- -   **How to run** clone the repo, enter the contracts folder and download the npm packages by running:
    ```bash
    npm install
    # or
    yarn add
    ```
    configure the hardhat.config file(default set to mumbai) then deploy to any chain of choice of using the commands
    ```bash
    npx hardhat run --network <your-network> scripts/deployGiveaway.js
    npx hardhat run --network <your-network> scripts/deployAirdrop.js
    ``` -->

-   ### **Wormhole**

    - Wormhole was a core technology used in the project that enabled the cross-chain capabilities of the dapp. Wormhole was used for sending message payloads and tokens across the various supported chains. Highlighted below are the various ways Wormhole was used in the development of the dapp.
    - Factory Contract: - Wormhole was used in sending messages to all factories to deploy a copy of the index contract. The crossChainDeployment function implements this functionality. the code is highlighted below:
    ```
    function crossChainDeployment(
        uint16 targetChain,
        address targetAddress,
        string memory name,
        string memory symbol,
        address _owner,
        address[] memory _assetContracts, 
        uint[] memory _assetRatio, 
        uint16[] memory _assetChains
    ) public payable {
        uint256 cost = quoteCrossChainDeployment(targetChain);
        // require(msg.value == cost, "not enough gas");
        wormholeRelayer.sendPayloadToEvm{value: cost}(
            targetChain,
            targetAddress,
            abi.encode(name, symbol, _owner, _assetContracts, _assetRatio, _assetChains), // payload
            0,
            GAS_LIMIT
        );
    }
    ```
    on reception of the wormhole message in the factory contract of the target chain the deployIndex function is called and the index is deployed - it is implemented with the below function:
    ```
    function receiveWormholeMessages(
        bytes memory payload,
        bytes[] memory, // additionalVaas
        bytes32, // address that called 'sendPayloadToEvm' (HelloWormhole contract address)
        uint16 sourceChain,
        bytes32 // unique identifier of delivery
    ) public payable {
        require(msg.sender == address(wormholeRelayer), "Only relayer allowed");

        // Parse the payload and do the corresponding actions!
        (string memory name, string memory symbol, address _owner, address[] memory _assetContracts, uint[] memory _assetRatio, uint16[] memory _assetChains) = abi.decode(
            payload,
            (string, string, address, address[], uint[], uint16[])
        );
        deployIndex(name, symbol, _owner, _assetContracts, _assetRatio, _assetChains);
    }
    ```
    - Router Contract - Wormhole messages was used in the Router contract for sending information to other router contracts on target chains in the sell off process, the function is called in the index contract like this:
    ```
    IRouter(routerAddress).crossChainRedeem{value: cost}(totalSupply(), targetIndex, amount, assetContracts[i], assetChains[i], msg.sender, purchaseToken);
    ```
    The crossChainRedeem function is implemented in the router contract this way:
    ```
    function crossChainRedeem(uint256 _totalSupply, address _targetIndex, uint256 amount, address _assetContract, uint16 targetChain, address receiver, address purchaseToken) public payable {
        uint256 cost = quoteCrossChainMessage(targetChain);
        require(msg.value == cost, "not enough gas");
        wormholeRelayer.sendPayloadToEvm{value: cost}(
            targetChain,
            routerAddresses[targetChain],
            abi.encode(_totalSupply, _assetContract, amount, _targetIndex, receiver, purchaseToken, chainId), // payload
            0,
            GAS_LIMIT
        );
    }
    ```
    When the Wormhole message is received on the router contract of the target chain it implements the receiveWormholeMessages function like this:
    ```
    function receiveWormholeMessages(bytes memory payload, bytes[] memory, bytes32, uint16 sourceChain, bytes32) public payable override {
        require(msg.sender == address(wormholeRelayer), "Only relayer allowed");

        // Parse the payload and do the corresponding actions!
        (uint256 totalSupply, address assetContract, uint256 userSupply, address targetIndex, address receiver, address outputTokenHomeAddress, uint16 sourceChainId) = abi.decode(
            payload,
            (uint256, address, uint256, address, address, address, uint16)
        );
        IFund(targetIndex).sale(userSupply, totalSupply, assetContract, receiver, outputTokenHomeAddress, sourceChainId);
    }
    ```
    The sale function on the index fund is called to execute the sale of an asset.
    - Index Fund Contract - Wormhole was used on the indexFund contract for the purchase of cross-chain tokens when the invest function is called, after the chain of an asset is determined, both payload(contains the asset contract address) and tokens is sent - see below
    ```
    sendTokenWithPayloadToEvm(
        assetChains[i],
        targetIndexContracts[i],
        payload,
        0,
        GAS_LIMIT,
        purchaseToken,
        tokenAmounts[i]
    );
    ```
    When a target index contract receives the payload and tokens it executes the purchase of an asset tokens in the overidden receivePayloadAndTokens function.
    ```
    function receivePayloadAndTokens(
        bytes memory payload,
        TokenReceived[] memory receivedTokens,
        bytes32, // sourceAddress
        uint16,
        bytes32 // deliveryHash
    ) internal override onlyWormholeRelayer {
        require(receivedTokens.length == 1, "Expected 1 token transfers");

        (address assetContract) = abi.decode(payload, (address));
        IERC20(receivedTokens[0].tokenAddress).approve(dexRouterAddress, receivedTokens[0].amount);
        dexRouter(dexRouterAddress).swapExactTokens(address(this), receivedTokens[0].amount, IERC20(receivedTokens[0].tokenAddress), IERC20(assetContract));
    }
    ```
    
> ## Deployment

-   ### **Deployment Addresses**

| <b><u>Contract</u></b>   | <b><u>Celo Testnet</u></b>                           | <b><u>Avalanche Testnet</u></b>                           |
| :----------------------- | :--------------------------------------------------- | :--------------------------------------------------- |
| **`Factory`**            |  0xc9B7979f7895676d4441F0dA619033ec896c6257                                                    |                                                      |
| **`Router`**             |                                                      |                                                      |
| **`Migrator`**           |                                                      |                                                      |
