<div align="center">

<img width="2088" alt="96893278-ebc67580-1460-11eb-9530-d5df3a3d65d0" src="https://user-images.githubusercontent.com/55017307/134509946-a8a006cc-2734-4575-9621-cd3680dabb11.png">

---

</div>

have you heard of a framework called hardhat? it's a new framework for smart contract that is getting super popular.

one of its key feature is a great debugging experience thanks to console log statement that you can use in your smart contract like in javascript.

---

before you learn hardhat I recommend to first learn truffle
fortunanlly I have a repo here: https://github.com/Fylls/learning-truffle

---

## Project setup

```bash
>>> npm init -y
>>> npm install -D hardhat
>>> npx hardhat

$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

Welcome to Hardhat v2.6.4
```

<br>

every time you run a command with hardhat you are actually running a task.
you can define tasks yourself but there are also a couple of default.

if you want to see all the tasks available you can type `npx hardhat` after creating the project.

hardhat also comes with an ecosystem of plugin that you can install in order to extend its functionalities.

( **npx** is something that comes installed with node.js it allow you to easily execute executable that are installed locally just for your project )

---

## Installing Plugins

```bash
npm install -D @nomiclabs/hardhat-ethers ethers @nomiclabs/hardhat-waffle ethereum-waffle chai
```

**waffle** is a package to test smart contract. it's not specific to hardhat so that's why we have to install a _plugin_ (@nomiclabs/hardhat-waffle) and the _library_ (ethereum-waffle) as well

we now need to edit `hardhat.config.js` and add the waffle plugin

```javascript
/**
 * @type import('hardhat/config').HardhatUserConfig
 */

require("@nomiclabs/hardhat-waffle")

module.exports = {
  solidity: "0.8.0",
}
```

---

## Compiling

make sure your contracts are in the `./contracts` folder

```bash
>>> npx hardhat compile
```

---

## Spin up Local Blockchain

same as ganache-cli

```bash
>>> npx hardhat node
```

---

## Testing

make sure your tests are in the `./test` folder

```bash
>>> npx hardhat test
```

---

## Smart contact debugging

`import "hardhat/console.sol";`

you can now use `console.log` in your solidity scripts

---

## Script for Deploying

ethers is injected by hardhat into `./scripts/deploy.js`

```javascript
async function main() {
  const [deployer] = await ethers.getSigners()
  console.log(`Deploying contracts with the account: ${deployer.address}`)

  const balance = await deployer.getBalance()
  console.log(`Account balance: ${balance.toString()}`)

  const Token = await ethers.getContractFactory("Token")
  const token = await Token.deploy()
  console.log(`Token address: ${token.address}`)
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error)
    process.exit(1)
  })
```

---

## Deploy on Local Blockchain

hardhat it's going to run an embedded instance of a local blockchain just in our memory. (same as ganache with truffle)

as soon as the command finishes to run, the local blockchain dies, so we're not going to be able to interact with this spot contract but doing it this way is still useful for making sure that the deployment script works fine.

default network ( --network http://localhost:8545 )

```bash
>>> npx hardhat run scripts/deploy.js

Deploying contracts with the account: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Account balance: 10000000000000000000000
Token address: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

---

## Deploy on Public Testnet

first we need to update our settings in `hardhat.config.js`

```javascript
/**
 * @type import('hardhat/config').HardhatUserConfig
 */
require("@nomiclabs/hardhat-waffle")

const INFURA_URL = ""
const PRIVATE_KEY = ""

module.exports = {
  solidity: "0.8.0",
  networks: {
    kovan: {
      url: INFURA_URL,
      accounts: [`0x${PRIVATE_KEY}`],
    },
  },
}
```

```bash
>>> npx hardhat run scripts/deploy.js --network kovan

Deploying contracts with the account: 0x30371B85a4e0FDA3A79223bB47d765dB51EC9E80
Account balance: 204000006527891940
Token address: 0x326152cd11cb8380FA8761B0ae4cB9373791343a
```

---

## Connect to Frontend

in truffle when you deploy your contract it produces a json file called the `contract artifact` that contains these
two information (ABI & contract address) but unfortunately for hardhat we are missing the address so we need to do a bit more work so here with the file system (fs)

```javascript
const fs = require("fs")

async function main() {
  const [deployer] = await ethers.getSigners()
  console.log(`Deploying contracts with the account: ${deployer.address}`)

  const balance = await deployer.getBalance()
  console.log(`Account balance: ${balance.toString()}`)

  const Token = await ethers.getContractFactory("Token")
  const token = await Token.deploy()
  console.log(`Token address: ${token.address}`)

  const data = {
    address: token.address,
    abi: JSON.parse(token.interface.format("json")),
  }
  fs.writeFileSync("frontend/src/Token.json", JSON.stringify(data))
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error)
    process.exit(1)
  })
```
