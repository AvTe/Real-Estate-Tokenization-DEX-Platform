# Real Estate Tokenization Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Solidity CI](https://github.com/KAMBALENGUNUNU/real-estate-tokenization/actions/workflows/solidity_ci.yml/badge.svg)](https://github.com/KAMBALENGUNUNU/real-estate-tokenization/actions/workflows/solidity_ci.yml)
<!-- Add other badges like build status, PRs welcome, etc. -->

This project is a decentralized real estate tokenization platform built on the Ethereum blockchain. It allows properties to be tokenized into ERC-721 Non-Fungible Tokens (NFTs), enabling features like fractional ownership (conceptually, though ERC-721 is whole ownership per token), dynamic valuation updates using Chainlink oracles, and rental income distribution to token holders.

## Table of Contents

*   [About The Project](#about-the-project)
    *   [Built With](#built-with)
    *   [Key Features](#key-features)
    *   [Contract Structure](#contract-structure)
*   [Getting Started](#getting-started)
    *   [Prerequisites](#prerequisites)
    *   [Installation](#installation)
*   [Usage](#usage)
*   [Roadmap](#roadmap)
*   [Contributing](#contributing)
*   [License](#license)
*   [Contact](#contact)
*   [Acknowledgments](#acknowledgments)

## About The Project

The Real Estate Tokenization Platform aims to bring liquidity and transparency to real estate investments through blockchain technology. By representing real-world properties as unique digital tokens (NFTs), this platform facilitates easier management, valuation, and revenue sharing.

### Built With

*   [Solidity](https://soliditylang.org/)
*   [OpenZeppelin Contracts](https://openzeppelin.com/contracts/)
    *   ERC721
    *   Ownable
    *   SafeMath
    *   ReentrancyGuard
*   [Chainlink](https://chain.link/)
*   [Hardhat](https://hardhat.org/) (or Truffle - adjust as per your project setup)

### Key Features

1.  **Tokenization of Properties**:
    *   Properties are tokenized into unique ERC-721 NFTs.
    *   Each token stores property-specific details like ID, valuation, oracle address, and rent collected.
2.  **Dynamic Valuation**:
    *   Utilizes Chainlink oracles (`updateValuation` function) to fetch real-time market data and update property valuations.
3.  **Rental Income Distribution**:
    *   The `distributeRent` function enables the distribution of rental income (sent as ETH to the contract) to the respective token holders.
    *   Protected by `nonReentrant` modifier to prevent reentrancy attacks.
4.  **Secure and Transparent Operations**:
    *   `SafeMath` is employed for arithmetic operations to prevent overflows/underflows.
    *   Events (`PropertyTokenized`, `RentDistributed`, `ValuationUpdated`) are emitted for major actions, enhancing on-chain transparency.
5.  **Ownership and Control**:
    *   Contract ownership is managed via OpenZeppelin's `Ownable`.
    *   Only the contract owner can mint new property tokens.
    *   The `_transfer` function is overridden with `onlyOwner` and `nonReentrant` modifiers for specific contract-initiated transfers, adding a layer of control.
6.  **ETH Reception**:
    *   A `receive()` fallback function allows the contract to accept direct Ether transfers, crucial for processes like rent collection.

### Contract Structure: `RealEstateTokenization.sol`

*   **`Property` Struct**: Defines the data structure for storing property details:
    *   `propertyId` (string): Unique identifier for the property.
    *   `valuation` (uint256): Current valuation of the property.
    *   `rentCollected` (uint256): Total rent collected for the property.
    *   `oracle` (address): Address of the Chainlink oracle for valuation updates.
*   **State Variables**:
    *   `properties` (mapping): Maps token IDs to `Property` structs.
    *   `nextTokenId` (uint256): Counter for minting new tokens.
*   **Events**:
    *   `PropertyTokenized(uint256 indexed tokenId, string propertyId, uint256 valuation)`
    *   `RentDistributed(uint256 indexed tokenId, uint256 amount)`
    *   `ValuationUpdated(uint256 indexed tokenId, uint256 newValuation)`
*   **Key Functions**:
    *   `constructor()`: Initializes the ERC721 token with a name ("RealEstateToken") and symbol ("RET").
    *   `tokenizeProperty(string memory _propertyId, uint256 _initialValuation, address _oracle)`: Mints a new property NFT (owner-only).
    *   `updateValuation(uint256 tokenId)`: Updates a property's valuation using its assigned Chainlink oracle.
    *   `distributeRent(uint256 tokenId)`: Distributes `msg.value` (ETH sent with the transaction) to the owner of the `tokenId`. *Note: The current implementation distributes the full `msg.value` to the owner of the specified `tokenId`. If the intent is to distribute among all token holders of a property or all token holders of the contract, the logic in `distributeRent` needs adjustment.*
    *   `getPropertyDetails(uint256 tokenId)`: Returns details of a specific tokenized property.
    *   `_transfer(address from, address to, uint256 tokenId)`: Internal transfer function, overridden to be `onlyOwner` and `nonReentrant`. This implies only the contract owner can initiate transfers through this specific internal function, which is unusual for standard ERC721 user-to-user transfers. Standard ERC721 `transferFrom` and `safeTransferFrom` would still be available for token holders.
    *   `tokenExists(uint256 tokenId)`: Checks if a token ID has been minted.
    *   `receive() external payable`: Fallback function to receive Ether.

## Getting Started

To get a local copy up and running follow these simple steps.

### Prerequisites

Ensure you have the following installed:
*   Node.js (e.g., v16 or later)
*   npm (Node Package Manager) or Yarn
*   A development environment for Ethereum smart contracts:
    *   Truffle: `npm install -g truffle`
    *   OR
    *   Hardhat: `npm install --save-dev hardhat`
*   Ganache CLI or Ganache UI for a local blockchain instance.
*   MetaMask browser extension for interacting with the DApp.

### Installation

1.  **Clone the repository**:
    ```bash
    git clone https://github.com/KAMBALENGUNUNU/real-estate-tokenization.git
    cd real-estate-tokenization
    ```
2.  **Install NPM packages**:
    ```bash
    npm install
    ```
    or if you use Yarn:
    ```bash
    yarn install
    ```
3.  **Compile Contracts** (Example using Hardhat):
    ```bash
    npx hardhat compile
    ```
    (Example using Truffle):
    ```bash
    truffle compile
    ```
4.  **Deploy Contracts** (Example using Hardhat with a script in `scripts/deploy.js`):
    ```bash
    npx hardhat run scripts/deploy.js --network <your_network_name>
    ```
    (Example using Truffle migrations):
    ```bash
    truffle migrate --network <your_network_name>
    ```
    *Note: You'll need to configure your `hardhat.config.js` or `truffle-config.js` with network details (e.g., for Ganache, Ropsten, Mainnet).*

## Usage

Once deployed, the contract owner can:
1.  **Tokenize a Property**: Call `tokenizeProperty` with the property ID, initial valuation, and the address of a suitable Chainlink Price Feed oracle.
2.  **Update Valuation**: Call `updateValuation` for a specific `tokenId` to refresh its valuation from the oracle.
3.  **Distribute Rent**: Send ETH to the contract by calling `distributeRent` for a specific `tokenId`. The owner of that token will receive the ETH.

Other users (token holders) can:
*   Transfer their property tokens using standard ERC721 transfer functions.
*   View property details using `getPropertyDetails`.

*For more detailed examples and interaction patterns, refer to the test scripts or consider building a front-end interface.*

## Roadmap

*   [ ] **Fractional Ownership**: Implement ERC-1155 or a custom solution for fractional ownership of properties.
*   [ ] **Marketplace Integration**: Allow listing and trading of property tokens.
*   [ ] **Governance**: Introduce a voting system for token holders on property management decisions.
*   [ ] **Enhanced Rent Distribution**: More sophisticated mechanisms for distributing rent to multiple stakeholders or based on shares.
*   [ ] **Legal Framework Integration**: Explore connections with legal frameworks for real-world asset tokenization.
*   [ ] **Advanced Oracle Usage**: Utilize more complex oracle data for property management (e.g., maintenance costs, occupancy rates).

See the [open issues](https://github.com/KAMBALENGUNUNU/real-estate-tokenization/issues) for a full list of proposed features (and known issues).

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1.  Fork the Project
2.  Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3.  Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4.  Push to the Branch (`git push origin feature/AmazingFeature`)
5.  Open a Pull Request

## License

Distributed under the MIT License. See `LICENSE` file for more information. (Note: You should add a LICENSE file to your repository if it's missing). The smart contract itself specifies `SPDX-License-Identifier: MIT`.

## Contact

Your Name / Organization - [@your_twitter](https://twitter.com/your_twitter) - email@example.com

Project Link: [https://github.com/KAMBALENGUNUNU/real-estate-tokenization](https://github.com/KAMBALENGUNUNU/real-estate-tokenization)

## Acknowledgments

*   OpenZeppelin
*   Chainlink
*   Ethereum Community
*   [README Template by Othneil Drew](https://github.com/othneildrew/Best-README-Template) (If you used one as inspiration)

```<!-- filepath: c:\Users\amitv\Downloads\real-estate-tokenization-master\real-estate-tokenization-master\README.md -->
# Real Estate Tokenization Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Solidity CI](https://github.com/KAMBALENGUNUNU/real-estate-tokenization/actions/workflows/solidity_ci.yml/badge.svg)](https://github.com/KAMBALENGUNUNU/real-estate-tokenization/actions/workflows/solidity_ci.yml)
<!-- Add other badges like build status, PRs welcome, etc. -->

This project is a decentralized real estate tokenization platform built on the Ethereum blockchain. It allows properties to be tokenized into ERC-721 Non-Fungible Tokens (NFTs), enabling features like fractional ownership (conceptually, though ERC-721 is whole ownership per token), dynamic valuation updates using Chainlink oracles, and rental income distribution to token holders.

## Table of Contents

*   [About The Project](#about-the-project)
    *   [Built With](#built-with)
    *   [Key Features](#key-features)
    *   [Contract Structure](#contract-structure)
*   [Getting Started](#getting-started)
    *   [Prerequisites](#prerequisites)
    *   [Installation](#installation)
*   [Usage](#usage)
*   [Roadmap](#roadmap)
*   [Contributing](#contributing)
*   [License](#license)
*   [Contact](#contact)
*   [Acknowledgments](#acknowledgments)

## About The Project

The Real Estate Tokenization Platform aims to bring liquidity and transparency to real estate investments through blockchain technology. By representing real-world properties as unique digital tokens (NFTs), this platform facilitates easier management, valuation, and revenue sharing.

### Built With

*   [Solidity](https://soliditylang.org/)
*   [OpenZeppelin Contracts](https://openzeppelin.com/contracts/)
    *   ERC721
    *   Ownable
    *   SafeMath
    *   ReentrancyGuard
*   [Chainlink](https://chain.link/)
*   [Hardhat](https://hardhat.org/) (or Truffle - adjust as per your project setup)

### Key Features

1.  **Tokenization of Properties**:
    *   Properties are tokenized into unique ERC-721 NFTs.
    *   Each token stores property-specific details like ID, valuation, oracle address, and rent collected.
2.  **Dynamic Valuation**:
    *   Utilizes Chainlink oracles (`updateValuation` function) to fetch real-time market data and update property valuations.
3.  **Rental Income Distribution**:
    *   The `distributeRent` function enables the distribution of rental income (sent as ETH to the contract) to the respective token holders.
    *   Protected by `nonReentrant` modifier to prevent reentrancy attacks.
4.  **Secure and Transparent Operations**:
    *   `SafeMath` is employed for arithmetic operations to prevent overflows/underflows.
    *   Events (`PropertyTokenized`, `RentDistributed`, `ValuationUpdated`) are emitted for major actions, enhancing on-chain transparency.
5.  **Ownership and Control**:
    *   Contract ownership is managed via OpenZeppelin's `Ownable`.
    *   Only the contract owner can mint new property tokens.
    *   The `_transfer` function is overridden with `onlyOwner` and `nonReentrant` modifiers for specific contract-initiated transfers, adding a layer of control.
6.  **ETH Reception**:
    *   A `receive()` fallback function allows the contract to accept direct Ether transfers, crucial for processes like rent collection.

### Contract Structure: `RealEstateTokenization.sol`

*   **`Property` Struct**: Defines the data structure for storing property details:
    *   `propertyId` (string): Unique identifier for the property.
    *   `valuation` (uint256): Current valuation of the property.
    *   `rentCollected` (uint256): Total rent collected for the property.
    *   `oracle` (address): Address of the Chainlink oracle for valuation updates.
*   **State Variables**:
    *   `properties` (mapping): Maps token IDs to `Property` structs.
    *   `nextTokenId` (uint256): Counter for minting new tokens.
*   **Events**:
    *   `PropertyTokenized(uint256 indexed tokenId, string propertyId, uint256 valuation)`
    *   `RentDistributed(uint256 indexed tokenId, uint256 amount)`
    *   `ValuationUpdated(uint256 indexed tokenId, uint256 newValuation)`
*   **Key Functions**:
    *   `constructor()`: Initializes the ERC721 token with a name ("RealEstateToken") and symbol ("RET").
    *   `tokenizeProperty(string memory _propertyId, uint256 _initialValuation, address _oracle)`: Mints a new property NFT (owner-only).
    *   `updateValuation(uint256 tokenId)`: Updates a property's valuation using its assigned Chainlink oracle.
    *   `distributeRent(uint256 tokenId)`: Distributes `msg.value` (ETH sent with the transaction) to the owner of the `tokenId`. *Note: The current implementation distributes the full `msg.value` to the owner of the specified `tokenId`. If the intent is to distribute among all token holders of a property or all token holders of the contract, the logic in `distributeRent` needs adjustment.*
    *   `getPropertyDetails(uint256 tokenId)`: Returns details of a specific tokenized property.
    *   `_transfer(address from, address to, uint256 tokenId)`: Internal transfer function, overridden to be `onlyOwner` and `nonReentrant`. This implies only the contract owner can initiate transfers through this specific internal function, which is unusual for standard ERC721 user-to-user transfers. Standard ERC721 `transferFrom` and `safeTransferFrom` would still be available for token holders.
    *   `tokenExists(uint256 tokenId)`: Checks if a token ID has been minted.
    *   `receive() external payable`: Fallback function to receive Ether.

## Getting Started

To get a local copy up and running follow these simple steps.

### Prerequisites

Ensure you have the following installed:
*   Node.js (e.g., v16 or later)
*   npm (Node Package Manager) or Yarn
*   A development environment for Ethereum smart contracts:
    *   Truffle: `npm install -g truffle`
    *   OR
    *   Hardhat: `npm install --save-dev hardhat`
*   Ganache CLI or Ganache UI for a local blockchain instance.
*   MetaMask browser extension for interacting with the DApp.

### Installation

1.  **Clone the repository**:
    ```bash
    git clone https://github.com/KAMBALENGUNUNU/real-estate-tokenization.git
    cd real-estate-tokenization
    ```
2.  **Install NPM packages**:
    ```bash
    npm install
    ```
    or if you use Yarn:
    ```bash
    yarn install
    ```
3.  **Compile Contracts** (Example using Hardhat):
    ```bash
    npx hardhat compile
    ```
    (Example using Truffle):
    ```bash
    truffle compile
    ```
4.  **Deploy Contracts** (Example using Hardhat with a script in `scripts/deploy.js`):
    ```bash
    npx hardhat run scripts/deploy.js --network <your_network_name>
    ```
    (Example using Truffle migrations):
    ```bash
    truffle migrate --network <your_network_name>
    ```
    *Note: You'll need to configure your `hardhat.config.js` or `truffle-config.js` with network details (e.g., for Ganache, Ropsten, Mainnet).*

## Usage

Once deployed, the contract owner can:
1.  **Tokenize a Property**: Call `tokenizeProperty` with the property ID, initial valuation, and the address of a suitable Chainlink Price Feed oracle.
2.  **Update Valuation**: Call `updateValuation` for a specific `tokenId` to refresh its valuation from the oracle.
3.  **Distribute Rent**: Send ETH to the contract by calling `distributeRent` for a specific `tokenId`. The owner of that token will receive the ETH.

Other users (token holders) can:
*   Transfer their property tokens using standard ERC721 transfer functions.
*   View property details using `getPropertyDetails`.

*For more detailed examples and interaction patterns, refer to the test scripts or consider building a front-end interface.*

## Roadmap

*   [ ] **Fractional Ownership**: Implement ERC-1155 or a custom solution for fractional ownership of properties.
*   [ ] **Marketplace Integration**: Allow listing and trading of property tokens.
*   [ ] **Governance**: Introduce a voting system for token holders on property management decisions.
*   [ ] **Enhanced Rent Distribution**: More sophisticated mechanisms for distributing rent to multiple stakeholders or based on shares.
*   [ ] **Legal Framework Integration**: Explore connections with legal frameworks for real-world asset tokenization.
*   [ ] **Advanced Oracle Usage**: Utilize more complex oracle data for property management (e.g., maintenance costs, occupancy rates).

See the [open issues](https://github.com/KAMBALENGUNUNU/real-estate-tokenization/issues) for a full list of proposed features (and known issues).

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1.  Fork the Project
2.  Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3.  Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4.  Push to the Branch (`git push origin feature/AmazingFeature`)
5.  Open a Pull Request

## License

Distributed under the MIT License. See `LICENSE` file for more information. (Note: You should add a LICENSE file to your repository if it's missing). The smart contract itself specifies `SPDX-License-Identifier: MIT`.

## Contact

Your Name / Organization - [@your_twitter](https://twitter.com/your_twitter) - email@example.com

Project Link: [https://github.com/KAMBALENGUNUNU/real-estate-tokenization](https://github.com/KAMBALENGUNUNU/real-estate-tokenization)

## Acknowledgments

*   OpenZeppelin
*   Chainlink
*   Ethereum Community
*   [README Template by Othneil Drew](https://github.com/othneildrew/Best-README-Template) (If you used one as inspiration)
