ArduinoMyLog4
=============

## Project ArduinoMyLog4 - (SCROLL - L2 NETWORK)

This project will store into an Layer2 Testnet Blockchain (Scroll Sepolia), the data of temperature provided by Arduino.

## Quick links and licensing references

- [Scroll seamlessly extends Ethereum’s capabilities through zero knowledge tech and EVM compatibility.](https://scroll.io/)
- [Foundry is a blazing fast, portable and modular toolkit for Ethereum application](https://getfoundry.sh/)
- [Arduino is an open-source electronics platform](https://www.arduino.cc/)
- [TMP36 library is a very simple Arduino library](https://github.com/Isaacr100/TMP36/)

## Tested for

* ARDUINO UNO BOARD with ARDUINO ETHERNET SHIELD (https://store.arduino.cc) equipped with temperature sensor TMP36 
(Note: any devices that provides data in json format could be included in this project)


## Notice
This software is experimental and a work in progress. Data inserted into blockchain will be accessible to anyone.
Under no circumstances should these files be used in relation to any critical system(s).
Use of these files is at your own risk.
THE SOFTWARE IS PROVIDED "AS IS". See license file LICENSE.md for details.

## Quick start

* For this project we will use the Layer 2 network Scroll Sepolia (which is a Testnet interacting with Ethereum) and that arduinomylog.ino file has been uploaded correctly into Arduino and that by calling the Arduino IP into a browser, we obtain a json similar to this:
```json
{"WeatherStationDC": [{"location": "Trento - Italy","temperature celsius": "15.40","temperature fahrenheit": "69.25"}]}
```
* Now we have to choose a blockchain, on which storing the temperature data retrieved from the sensor based on the JSON generated by Arduino.

  For this example is used the TESTNET version of Scroll BLOCKCHAIN ( https://sepolia.scroll.io/ );
  
  if you are not familiar with how a blockchain works, it is recommended to read more information here:
  
  [https://docs.scroll.io/en/home/](https://docs.scroll.io/en/home/).
  
 To check the transactions of Layer2 you can use the L2 Block explorer ( https://sepolia.scrollscan.com/ ), while to verify the related rollup it is possible to have a check here

 https://sepolia.scroll.io/rollupscan?page=1&per_page=10 
 
 and finally on the Layer 1

 https://sepolia.etherscan.io/

For this example we will use EOA "0xa67a79cF9EaD85879e2d15238707aFC0a2f45EAa" on SCROLL SEPOLIA TESTNET.

* Let's start - Get Docker image of Foundry:
```shell
docker pull ghcr.io/foundry-rs/foundry:latest
docker tag ghcr.io/foundry-rs/foundry:latest foundry:v0.2.0
# verify
docker run --rm -it --name foundry_test foundry:v0.2.0 -c "forge --version"
```

* Initialize the project ( more info https://book.getfoundry.sh/ ):
```shell
mkdir -p ~/lanscrollsepolia_box/box

docker run \
--rm \
-it \
-d \
--name foundry_initiate \
-v ~/lanscrollsepolia_box/box:/app \
foundry:v0.2.0 \
"forge init --no-git /app"

# Install libraries and create smart contract

docker run \
--rm \
-it \
-d \
--name foundry_inst_libraries \
-v ~/lanscrollsepolia_box/box:/app \
foundry:v0.2.0 \
"forge install OpenZeppelin/openzeppelin-contracts --no-git --root /app"

# verify the new directories

tree ~/lanscrollsepolia_box/box/lib -d -L 1

OUTPUT:

├── forge-std
└── openzeppelin-contracts

# create smart contract

sudo rm -f ~/lanscrollsepolia_box/box/src/Counter.sol
sudo rm -f ~/lanscrollsepolia_box/box/script/Counter.s.sol
sudo rm -f ~/lanscrollsepolia_box/box/test/Counter.t.sol
sudo touch ~/lanscrollsepolia_box/box/src/box.sol
```

* Check the solidity file:

cat ~/lanscrollsepolia_box/box/src/box.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;
import "@openzeppelin/contracts/access/Ownable.sol";
contract Box is Ownable {
   constructor() Ownable(msg.sender) {}
   uint256 private _value;
    event ValueChanged(uint256 value);
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```
* Compile the contract:
```shell
docker run \
--rm \
-it \
-d \
--name foundry_build \
-v ~/lanscrollsepolia_box/box:/app \
--entrypoint sh \
-w /app \
foundry:v0.2.0 \
-c "forge build"
```

* Deploy smart contract:

Note: You must have some funds into your EOA before deploy the Smart Contract.

In this case we can check the EOA's balance of this example here:

https://sepolia.scrollscan.com/address/0xa67a79cF9EaD85879e2d15238707aFC0a2f45EAa

```shell
cd ~/lanscrollsepolia_box/box

docker run \
--rm \
-it \
-d \
--name foundry_deploy \
-v ~/lanscrollsepolia_box/box:/app \
--entrypoint sh \
-w /app \
foundry:v0.2.0 \
-c "forge create --rpc-url=https://sepolia-rpc.scroll.io --private-key [PrivateKey] src/box.sol:Box"
```
OUTPUT:

Deployer: 0xa67a79cF9EaD85879e2d15238707aFC0a2f45EAa

Deployed to: 0x6A2C5E2B519b07E6939363f44d9dF4E23af73b86

* Interact with smart contract:
```shell
docker run \
--rm \
-it \
--name foundry_deploy \
-v ~/lanscrollsepolia_box/box:/app \
--entrypoint sh \
-w /app \
foundry:v0.2.0 \
-c "forge inspect Box storage-layout --pretty"
```
OUTPUT:

```shell
| Name   | Type    | Slot | Offset | Bytes | Contract        |
|--------|---------|------|--------|-------|-----------------|
| _owner | address | 0    | 0      | 20    | src/box.sol:Box |
| _value | uint256 | 1    | 0      | 32    | src/box.sol:Box |
```

* Insert a value into smart contract 0x6A2C5E2B519b07E6939363f44d9dF4E23af73b86 :
```shell
docker run \
--rm \
-it \
-d \
--name foundry_deploy \
-v ~/lanscrollsepolia_box/box:/app \
--entrypoint sh \
-w /app \
foundry:v0.2.0 \
-c "cast send 0x6A2C5E2B519b07E6939363f44d9dF4E23af73b86 'store(uint256)' '1578' --rpc-url https://sepolia-rpc.scroll.io --private-key [PrivateKey]"
```

* Now retrieve the value of temperature from the smart contract :
```shell
docker run \
--rm \
-it \
--name foundry_call_retrieve \
-v ~/lanscrollsepolia_box/box:/app \
--entrypoint sh \
-w /app \
foundry:v0.2.0 \
-c "cast call 0x6A2C5E2B519b07E6939363f44d9dF4E23af73b86 'retrieve()(uint256)' --rpc-url https://sepolia-rpc.scroll.io"
```
OUTPUT:

1578 -------👉-------> is 15.78 °C

Note: Scroll blockchain, like many other blockchains including Ethereum, uses an integer representation to handle data. This means that the numbers are represented without the decimal point.


* Monitor the data in a quick way:

See your data on blockchain (they are accessible to anyone) by visiting a blockchain explorer;

in this case visit:

https://sepolia.scrollscan.com/address/0x6A2C5E2B519b07E6939363f44d9dF4E23af73b86#events

and then, in the drop-down menu, 👉 change from "Hex" to "Number"

* Optional: insert the code into crontab

Now you can now insert the above code into crontab to fetch the temperature data json coming from Arduino and insert IT automatically into blockchain. For example you will have an example log similar to this one:  
```shell
🕑 2024-03-25 16:30:00 CET
🟩 Poweron container:
container_fndr
🕑 2024-03-25 16:31:01 CET
⛏ Write temperature into blockchain:
Temperature value: 1578
🕑 2024-03-25 16:33:01 CET
🟥 Poweroff container:
container_fndr




















