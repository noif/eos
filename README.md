# Eos

[![Build Status](https://travis-ci.org/EOSIO/eos.svg?branch=master)](https://travis-ci.org/EOSIO/eos)

Welcome to the EOS.IO source code repository!

## Getting Started
The following instructions overview the process of getting the software, building it, running a simple test network that produces blocks, account creation and uploading a sample contract to the blockchain.

## Setting up a build/development environment
This project is written primarily in C++14 and uses CMake as its build system. An up-to-date Clang and the latest version of CMake is recommended.

Dependencies:

* Clang 4.0.0
* CMake 3.5.1
* Boost 1.64
* OpenSSL
* LLVM 4.0
* [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git)
* [binaryen](https://github.com/WebAssembly/binaryen.git)

### Clean install Ubuntu 16.10

Install the development toolkit:

```commandline
sudo apt-get update
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
sudo apt-get install clang-4.0 lldb-4.0 cmake make \
                     libbz2-dev libssl-dev libgmp3-dev \
                     autotools-dev build-essential \
                     libbz2-dev libicu-dev python-dev \
                     autoconf libtool git
```

Install Boost 1.64:

```commandline
cd ~
wget -c 'https://sourceforge.net/projects/boost/files/boost/1.64.0/boost_1_64_0.tar.bz2/download' -O boost_1.64.0.tar.bz2
tar xjf boost_1.64.0.tar.bz2
cd boost_1_64_0/
echo "export BOOST_ROOT=$HOME/opt/boost_1_64_0" >> ~/.bash_profile
source ~/.bash_profile
./bootstrap.sh "--prefix=$BOOST_ROOT"
./b2 install
source ~/.bash_profile
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):
        
```commandline
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make
sudo make install
```

Also, to use the WASM compiler, eos has an external dependency on [binaryen](https://github.com/WebAssembly/binaryen.git):

```commandline
cd ~
git clone https://github.com/WebAssembly/binaryen.git
cd ~/binaryen
git checkout tags/1.37.14
cmake . && make

```

Add BINARYEN_ROOT to your .bash_profile:

```commandline
echo "export BINARYEN_ROOT=~/binaryen" >> ~/.bash_profile
source ~/.bash_profile
```

By default LLVM and clang do not include the WASM build target, so you will have to build it yourself:

```commandline
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j4 install
```

### macOS Sierra 10.12.6

macOS additional Dependencies:

* Brew
* Newest XCode

Upgrade your XCode to the newest version:

```commandline
xcode-select --install
```

Install homebrew:

```commandline
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Install the dependencies:

```commandline
brew update
brew install git automake libtool boost openssl llvm
```

Install [secp256k1-zkp (Cryptonomex branch)](https://github.com/cryptonomex/secp256k1-zkp.git):
        
```commandline
cd ~
git clone https://github.com/cryptonomex/secp256k1-zkp.git
cd secp256k1-zkp
./autogen.sh
./configure
make
sudo make install
```

Install [binaryen v1.37.14](https://github.com/WebAssembly/binaryen.git):

```commandline
cd ~
git clone https://github.com/WebAssembly/binaryen.git
cd ~/binaryen
git checkout tags/1.37.14
cmake . && make
```

Add BINARYEN_ROOT to your .bash_profile:

```commandline
echo "export BINARYEN_ROOT=~/binaryen" >> ~/.bash_profile
source ~/.bash_profile
```


Build LLVM and clang for WASM:

```commandline
mkdir  ~/wasm-compiler
cd ~/wasm-compiler
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/llvm.git
cd llvm/tools
git clone --depth 1 --single-branch --branch release_40 https://github.com/llvm-mirror/clang.git
cd ..
mkdir build
cd build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=.. -DLLVM_TARGETS_TO_BUILD= -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DCMAKE_BUILD_TYPE=Release ../
make -j4 install
```

Add WASM_LLVM_CONFIG and LLVM_DIR to your .bash_profile:

```commandline
echo "export WASM_LLVM_CONFIG=~/wasm-compiler/llvm/bin/llvm-config" >> ~/.bash_profile
echo "export LLVM_DIR=/usr/local/Cellar/llvm/4.0.1/lib/cmake/llvm" >> ~/.bash_profile
source ~/.bash_profile
```

## Building and running a node

### Getting the code
To download all of the code, download Eos and a recursion or two of submodules. The easiest way to get all of this is to do a recursive clone:

`git clone https://github.com/eosio/eos --recursive`

If a repo is cloned without the `--recursive` flag, the submodules can be retrieved after the fact by running this command from within the repo:

`git submodule update --init --recursive`

### Using the WASM compiler to perform a full build of the project

The WASM_LLVM_CONFIG environment variable is used to find our recently built WASM compiler.
This is needed to compile the example contracts inside eos/contracts folder and their respective tests.

```commandline
cd ~
git clone https://github.com/eosio/eos --recursive
mkdir -p ~/eos/build && cd ~/eos/build
cmake -DBINARYEN_BIN=~/binaryen/bin ..
make -j4
```

Out-of-source builds are also supported. To override clang's default choice in compiler, add these flags to the CMake command:

`-DCMAKE_CXX_COMPILER=/path/to/c++ -DCMAKE_C_COMPILER=/path/to/cc`

For a debug build, add `-DCMAKE_BUILD_TYPE=Debug`. Other common build types include `Release` and `RelWithDebInfo`.

To run the test suite after building, run the `chain_test` executable in the `tests` folder.

### Creating and launching a single-node testnet

After successfully building the project, the `eosd` binary should be present in the `programs/eosd` directory. Go ahead and run `eosd` -- it will probably exit with an error, but if not, close it immediately with Ctrl-C. Note that `eosd` will have created a directory named `data-dir` containing the default configuration (`config.ini`) and some other internals. This default data storage path can be overridden by passing `--data-dir /path/to/data` to `eosd`.

Edit the `config.ini` file, adding the following settings to the defaults already in place:

```
# Load the testnet genesis state, which creates some initial block producers with the default key
genesis-json = /path/to/eos/source/genesis.json
# Enable production on a stale chain, since a single-node test chain is pretty much always stale
enable-stale-production = true
# Enable block production with the testnet producers
producer-name = inita
producer-name = initb
producer-name = initc
producer-name = initd
producer-name = inite
producer-name = initf
producer-name = initg
producer-name = inith
producer-name = initi
producer-name = initj
producer-name = initk
producer-name = initl
producer-name = initm
producer-name = initn
producer-name = inito
producer-name = initp
producer-name = initq
producer-name = initr
producer-name = inits
producer-name = initt
producer-name = initu
# Load the block producer plugin, so we can produce blocks
plugin = eos::producer_plugin
# As well as API and HTTP plugins
plugin = eos::chain_api_plugin
plugin = eos::http_plugin
```

Now it should be possible to run `eosd` and see it begin producing blocks. At present, the P2P code is not implemented, so only single-node configurations are possible. When the P2P networking is implemented, these instructions will be updated to show how to create an example multi-node testnet.

When running `eosd` you should get log messages similar to below. It means the blocks are successfully produced.

```
1575001ms thread-0   chain_controller.cpp:235      _push_block          ] initm #1 @2017-09-04T04:26:15  | 0 trx, 0 pending, exectime_ms=0
1575001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initm generated block #1 @ 2017-09-04T04:26:15 with 0 trxs  0 pending
1578001ms thread-0   chain_controller.cpp:235      _push_block          ] initc #2 @2017-09-04T04:26:18  | 0 trx, 0 pending, exectime_ms=0
1578001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initc generated block #2 @ 2017-09-04T04:26:18 with 0 trxs  0 pending
1581001ms thread-0   chain_controller.cpp:235      _push_block          ] initd #3 @2017-09-04T04:26:21  | 0 trx, 0 pending, exectime_ms=0
1581001ms thread-0   producer_plugin.cpp:207       block_production_loo ] initd generated block #3 @ 2017-09-04T04:26:21 with 0 trxs  0 pending
1584000ms thread-0   chain_controller.cpp:235      _push_block          ] inite #4 @2017-09-04T04:26:24  | 0 trx, 0 pending, exectime_ms=0
1584000ms thread-0   producer_plugin.cpp:207       block_production_loo ] inite generated block #4 @ 2017-09-04T04:26:24 with 0 trxs  0 pending
1587000ms thread-0   chain_controller.cpp:235      _push_block          ] initf #5 @2017-09-04T04:26:27  | 0 trx, 0 pending, exectime_ms=0
```

## Accounts and smart contracts

EOS comes with example contracts that can be uploaded and run for testing purposes. To upload and test them, please follow the steps below.

### Create accounts for your smart contracts

To publish sample smart contracts you need to create accounts for them.

At the moment for the testing purposes you need to run `eosd` with `--skip-transaction-signatures` flag to successfully create accounts and run transactions.

Run the node:

```commandline
cd ~/eos/build/programs/eosd/
./eosd --skip-transaction-signatures
```

First, generate public/private key pairs for the `owner_key` and `active_key`. We will need them to create an account:

```commandline
cd ~/eos/build/programs/eosc/
./eosc create key
./eosc create key
```

You will get two pairs of a public and private key:

```
Private key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Public key:  XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

Save the values for future reference.

Run `create` command where `PUBLIC_KEY_1` and `PUBLIC_KEY_2` are the values generated by the `create key` command:

```commandline
./eosc create account inita currency PUBLIC_KEY_1 PUBLIC_KEY_2 
```

You should get a json response back with a transaction ID confirming it was executed successfully.

Check that account was successfully created: 

```commandline
./eosc get account currency
```

You should get a response similar to this:

```json
{
  "name": "currency",
  "eos_balance": 0,
  "staked_balance": 1,
  "unstaking_balance": 0,
  "last_unstaking_time": "2106-02-07T06:28:15"
}
```

### Upload sample contract

Before uploading a contract, we can verify that there is no current contract:

```commandline
./eosc get code currency 
code hash: 0000000000000000000000000000000000000000000000000000000000000000
```

With an account for a contract created, you can upload a sample contract:

```commandline
./eosc set contract currency ../../../contracts/currency/currency.wast ../../../contracts/currency/currency.abi
```

As a response you should get a json with a `transaction_id` field. Your contract was successfully uploaded!

We can also verify that the code has been set:

```commandline
./eosc get code currency
code hash: 9b9db1a7940503a88535517049e64467a6e8f4e9e03af15e9968ec89dd794975
```

Next we can verify that the currency contract has the proper initial balance:

```commandline
./eosc get table currency currency account
{
  "rows": [{
     "account": "account",
     "balance": 1000000000
     }
  ],
  "more": false
}
```

### Push a message to a sample contract

To send a message to a contract we need to create a new user account who will be sending the message.

Firstly, generate the keys for the account:

```commandline
cd ~/eos/build/programs/eosc/
./eosc create key
./eosc create key
```

And then create the `tester` account:

```commandline
./eosc create account inita tester PUBLIC_USER_KEY_1 PUBLIC_USER_KEY_2 
```

After this we can send a message to the contract:

```commandline
./eosc push message currency transfer '{"from":"currency","to":"inita","amount":50}' --scope currency,inita --permission currency@active
```

As a confirmation of a successfully submitted transaction you will get a json with a `transaction_id` field.

### Reading Currency Contract Balance

```commandline
./eosc get table inita currency account
{
  "rows": [{
      "account": "account",
      "balance": 50 
       }
    ],
  "more": false
}
./eosc get table currency currency account
{
  "rows": [{
      "account": "account",
      "balance": 999999950
    }
  ],
  "more": false
}
```



## Run eos in docker

Simple and fast setup of EOS on Docker is also available. Firstly, install dependencies:

 - [Docker](https://docs.docker.com)
 - [Docker-compose](https://github.com/docker/compose)
 - [Docker-volumes](https://github.com/cpuguy83/docker-volumes)

Build eos image

```
git clone https://github.com/EOSIO/eos.git --recursive
cd eos
cp genesis.json Docker 
docker build -t eosio/eos -f Docker/Dockerfile .
```

We recommend 6GB+ of memory allocated to Docker to successfully build the image.

Starting the Docker this can be tested from container's host machine:

```
sudo rm -rf /data/store/eos # options 
sudo mkdir -p /data/store/eos
docker-compose -f Docker/docker-compose.yml up
```

Get chain info:

```
curl http://127.0.0.1:8888/v1/chain/get_info
```

### Run contract in docker example

You can run the `eosc` commands via `docker exec` command. For example:

```
docker exec docker_eos_1 eosc contract exchange contracts/exchange/exchange.wast contracts/exchange/exchange.abi
```
