[TOC]

# About the Stack
Before you start using the tools you just installed, it's a good idea to understand each of the components and how they interact with one another.

nodeos (node + eos = nodeos) - the core EOSIO node daemon that can be configured with plugins to run a node. Example uses are block production, dedicated API endpoints, and local development.
cleos (cli + eos = cleos) - command line interface to interact with the blockchain and to manage wallets
keosd (key + eos = keosd) - component that securely stores EOSIO keys in wallets.
eosio-cpp - Part of eosio.cdt, it compiles C++ code to WASM and can generate ABIs


# 1. DEVELOPMENT ENVIRONMENT

## 1.1 Start Your Node & Setup
### 1.1.1 Get the docker image

~~~
docker pull eosio/eos:v1.4.0
~~~

### 1.1.2 Boot Node and Wallet

~~~

docker run --name eosio   --publish 8888:8888   --publish 127.0.0.1:5555:5555   --volume /c/Users/lencs/Desktop/Projets/Eosdemo/contracts:/contracts  --detach   eosio/eos   /bin/bash -c   "keosd --http-server-address=0.0.0.0:5555 & exec nodeos -e -p eosio --plugin eosio::producer_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_plugin --plugin eosio::history_api_plugin --plugin eosio::http_plugin -d /mnt/dev/data --config-dir /mnt/dev/config --http-server-address=0.0.0.0:8888 --access-control-allow-origin=* --contracts-console --http-validate-host=false --filter-on='*'"
~~~


These settings accomplish the following:

Forward port 8888 and 5555 to host machine
Alias a work volume on your local drive to the docker container.
Run the Nodeos startup in bash. This command loads all the basic plugins, set the server address, enable CORS and add some contract debugging.
Enable CORS with no restrictions (*)

### 1.1.3 Check the installation

#### 1.1.3.1 Check that Nodeos is Producing Blocks

~~~
docker logs --tail 10 eosio
~~~

#### 1.1.3.2 Check the Wallet

~~~
docker exec -it eosio bash

cleos --wallet-url http://127.0.0.1:5555 wallet list keys
~~~

Now that keosd is running correctly, type exit and then press enter to leave the keosd shell. From this point forward, you won't need to enter the containers with bash, and you'll be executing commands from your local system (Linux or Mac)

#### 1.1.3.3 Check Nodeos endpoints

~~~
curl http://localhost:7777/v1/chain/get_info
~~~

### 1.1.4 Aliasing Cleos


~~~
alias cleos='docker exec -it eosio /opt/eosio/bin/cleos --url http://127.0.0.1:8888 --wallet-url http://127.0.0.1:5555'
~~~

### 1.1.5 Take Note of Useful Docker Commands


~~~
docker start eosio
docker stop eosio
docker exec -it eosio bash
docker rm eosio
~~~

## 1.2 Contract Development Toolkit

~~~
git clone --recursive https://github.com/eosio/eosio.cdt --branch v1.3.2 --single-branch
cd eosio.cdt

wget https://github.com/eosio/eosio.cdt/releases/download/v1.3.2/eosio.cdt-1.3.2.x86_64.deb
sudo apt install ./eosio.cdt-1.3.2.x86_64.deb
~~~


## 1.3 Create Development Wallet

### 1.3.1 Create a Wallet

~~~
cleos wallet create --to-console
~~~

    use --to-file so your wallet password is not in your bash history

cleos will return a password,

    Save password to use in the future to unlock this wallet.
    Without password imported keys will not be retrievable.

PW5JQDkjiugXDxEttBQpmYUcv12UKgAN6FtkNNaD9A8jJbS5YcXDa

### 1.3.2 Open the Wallet


    cleos wallet open


### 1.3.3 Unlock it

    cleos wallet unlock
PW5JQDkjiugXDxEttBQpmYUcv12UKgAN6FtkNNaD9A8jJbS5YcXDa

### 1.3.4 Import keys into your wallet

    cleos wallet create_key

Created new private key with a public key of: "
EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA

### 1.3.5 Import the Development Key

cleos wallet import

    You'll be prompted for a private key, enter the eosio development key provided below
5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3

    private key: imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

Wonderful, you now have a default wallet unlocked and loaded with a key, and are ready to proceed.



## 1.4 Create Test Accounts

在keys上建立新的账户

~~~
cleos create account eosio bob EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA
cleos create account eosio alice EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA
~~~

    executed transaction: 7b6408df6f7bad81915192d67deb61853217b7ad5d805ce326acee61ca1b9887  200 bytes  440 us
    #         eosio <= eosio::newaccount            {"creator":"eosio","name":"bob","owner":{"threshold":1,"keys":[ {"key":"EOS6L6hTF9N83aekjpdfpkvBd19Ph...
    warning: transaction executed locally, but may not be confirmed by the network yet         ]


-u http://127.0.0.1:5555

    Using Different Keys for Active/Owner on a PRODUCTION Network
EOSIO has a unique authorization structure that has added security for you account. You can minimize the exposure of your account by keeping the owner key cold, while using the key associated with your active permission. This way, if your active key were every compromised, you could regain control over your account with your owner key.

~~~
nodeos --replay-blockchain --hard-replay-blockchain
~~~

## 1.5 docker WSL

~~~
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get install ca-certificates
sudo apt-get install curl
sudo apt-get install software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
sudo apt-get update
sudo apt-get install docker-ce

export DOCKER_HOST=tcp://localhost:2375

sudo apt install python-pip
sudo pip install docker-compose
docker-compose version

sudo nano /etc/wsl.conf

[automount]
enabled = true
root = /
options = "metadata,umask=22,fmask=11"
mountFsTab = false
~~~


# 2. SMART CONTRACT DEVELOPMENT

~~~
cd /c/Users/lencs/Desktop/Projets/Eosdemo/contracts
~~~

## 2.1 Hello World!

~~~c++
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

using namespace eosio;

class hello : public contract {
  public:
      using contract::contract;

      [[eosio::action]] 
      void hi( name user ) {
         print( "Hello, ", name{user});
      }
};
EOSIO_DISPATCH( hello, (hi))
~~~

You can compile your code to web assembly (.wasm) as follows:

~~~
eosio-cpp -o hello.wasm hello.cpp --abigen

cleos --wallet-url http://127.0.0.1:5555 wallet list keys


cleos create account eosio hello EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA -p eosio@active
~~~

    root@4a180d42b1bd:/# cleos create account eosio hello EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA -p  eosio@active
    executed transaction: 0e7f171b1a8d423b4b9f0fe14c1d998054a271f9689fa4528281105be334cff0  200 bytes  435 us
    #         eosio <= eosio::newaccount            {"creator":"eosio","name":"hello","owner":{"threshold":1,"keys":[   {"key":"EOS6UqHxpYp2K9o8qtXLPd62i7j...
    warning: transaction executed locally, but may not be confirmed by the network yet         ]

~~~
cd /contracts

cleos set contract hello /contracts/hello -p hello@active
~~~

    root@71b4a5b68a55:/contracts/hello# cleos set contract hello /contracts/hello -p hello@active
    Reading WASM from /contracts/hello/hello.wasm...
    Publishing contract...
    executed transaction: 1b44a063ddccc337401b01a9932d8d865391b4c5b51a8ed3d1ae080456340192  1432 bytes  22073 us
    #         eosio <= eosio::setcode               {"account":"hello","vmtype":0,"vmversion":0,    "code":"0061736d0100000001390b60027f7e006000017f60027f7f...
    #         eosio <= eosio::setabi                {"account":"hello", "abi":"0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d650100000000...
    warning: transaction executed locally, but may not be confirmed by the network yet         ]

Great! Now the contract is set, push an action to it.

~~~
cleos push action hello hi '["bob"]' -p bob@active
~~~

    executed transaction:   25fd957b8992aff88a9834cb01a8641c5d9b9dfcd706eefe61b5e2dffbb6c17f      104 bytes  15608 us
    #         hello <= hello::hi                    {"user":"bob"}
    >> Hello, bob


add
    require_auth(user);

Recompile the contract

eosio-cpp -o hello.wasm hello.cpp --abigen

And then update it

~~~
docker exec -it eosio bash
cd /contracts/hello
cleos set contract hello /contracts/hello -p hello@active

cleos push action hello hi '["bob"]' -p bob@active
~~~


## 2.2Deploy, Issue and Transfer Tokens

### preparation
~~~
git clone https://github.com/EOSIO/eosio.contracts --branch v1.4.0 --single-branch

cd eosio.contracts/eosio.token

cleos create account eosio eosio.token EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

eosio-cpp -I include -o eosio.token.wasm src/eosio.token.cpp --abigen

cleos set contract eosio.token /contracts/eosio.contracts/eosio.token --abi eosio.token.abi -p eosio.token@active
~~~

### Create the Token

To create a new token call the create(...) action with the proper arguments. This action accepts 1 argument, it's a symbol_name type composed of two pieces of data, a maximum supply float and a symbol_name in capitalized alpha characters only, for example "1.0000 SYM". The issuer will be the one with authority to call issue and or perform other actions such as freezing, recalling, and whitelisting of owners.

Below is the concise way to call this method, using positional arguments:

~~~
cleos push action eosio.token create '[ "eosio", "1000000000.0000 SYS"]' -p eosio.token@active
cleos push action eosio.token create '{"issuer":"eosio", "maximum_supply":"1000000000.0000 SYS"}' -p eosio.token@active
~~~


    executed transaction: 28c310ff4801ce5809cbd2b30c0c45b8ade4d341bf28ce36a321194470f038e5  120 bytes  440 us
    #   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}
    warning: transaction executed locally, but may not be confirmed by the network yet         ]

### Issue Tokens

~~~
cleos push action eosio.token issue '[ "alice", "100.0000 SYS", "memo" ]' -p eosio@active
~~~


    executed transaction: 5c24546282f990cd73326c76e3118f1fbb2a0c589894d55bc3a48a1a5c45bef3  128 bytes  1949 us
    #   eosio.token <= eosio.token::issue           {"to":"alice","quantity":"100.0000 SYS","memo":"memo"}
    #   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"alice","quantity":"100.0000 SYS""memo":"memo"}
    #         eosio <= eosio.token::transfer        {"from":"eosio","to":"alice","quantity":"100.0000 SYS""memo":"memo"}
    #         alice <= eosio.token::transfer        {"from":"eosio","to":"alice","quantity":"100.0000 SYS""memo":"memo"}
    warning: transaction executed locally, but may not be confirmed by the network yet         ]

~~~
cleos push action eosio.token issue '["alice", "100.0000 SYS", "memo"]' -p eosio@active -d -j
~~~

~~~json
{
  "expiration": "2018-11-02T10:23:56",
  "ref_block_num": 4926,
  "ref_block_prefix": 3235143451,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "eosio.token",
      "name": "issue",
      "authorization": [{
          "actor": "eosio",
          "permission": "active"
        }
      ],
      "data": "0000000000855c3440420f00000000000453595300000000046d656d6f"
    }
  ],
  "transaction_extensions": [],
  "signatures": [
    "SIG_K1_JzpStCThBhzAwwTyNf6M57NtdL3SsPVg7SPKpcDrktcooKgdghxRZj4ZEirQKEGohVYxsRkLMXXh2yz1FxUhDbSTsx7dKt"
  ],
  "context_free_data": []
}
~~~

### Transfer Tokens

cleos push action eosio.token transfer '[ "alice", "bob", "25.0000 SYS", "m" ]' -p alice@active

    executed transaction: f6021942631664381fa1dca747650da0f599d6169e2c84a247ee89f397002b38  128 bytes  1067 us
    #   eosio.token <= eosio.token::transfer        {"from":"alice","to":"bob","quantity":"25.0000 SYS","memo":"m"}
    #         alice <= eosio.token::transfer        {"from":"alice","to":"bob","quantity":"25.0000 SYS","memo":"m"}
    #           bob <= eosio.token::transfer        {"from":"alice","to":"bob","quantity":"25.0000 SYS","memo":"m"}
    warning: transaction executed locally, but may not be confirmed by the network yet         ]


cleos get currency balance eosio.token alice SYS


## 2.3 Understanding ABI Files

The Application Binary Interface (ABI) is a JSON-based description on how to convert user actions between their JSON and Binary representations. The ABI also describes how to convert the database state to/from JSON. Once you have described your contract via an ABI then developers and users will be able to interact with your contract seamlessly via JSON.


## 2.4 Data Persistence

~~~
mkdir addressbook
cd addressbook
~~~

Write an Extended Standard Class and Include EOSIO

~~~c++
#include <eosiolib/eosio.hpp>

using namespace eosio;

class [[eosio::contract]] addressbook : public eosio::contract {
  public:
       
  private: 
  
};
~~~

~~~
eosio-cpp -o addressbook.wasm adr.cpp --abigen
cleos create account eosio addressbook EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA

cleos set contract addressbook /contracts/addressbook

cleos push action addressbook upsert '["alice", "alice", "liddell", "123 drink me way", "wonderland", "amsterdam"]' -p alice@active
cleos push action addressbook upsert '["bob", "bob", "is a loser", "doesnt exist", "somewhere", "someplace"]' -p alice@active
cleos get table addressbook addressbook people -k alice
cleos push action addressbook erase '["alice"]' -p alice@active
~~~



## 2.5 Adding Inline Actions
how to construct actions, and send those actions from within a contract.

### Step 1: Adding eosio.code to permissions

~~~

cleos set account permission addressbook active '{"threshold": 1,"keys": [{"key": "EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA","weight": 1}], "accounts": [{"permission":{"actor":"addressbook","permission":"eosio.code"},"weight":1}]}' -p addressbook@owner
~~~

The eosio.code authority is a pseudo authority implemented to enhance security, and enable contracts to execute inline actions.




~~~ c++

#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

using namespace eosio;

class [[eosio::contract]] addressbook : public eosio::contract {

public:
  using contract::contract;
  
  addressbook(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

  [[eosio::action]]
  void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
    require_auth( user );

    address_index addresses(_code, _code.value);

    auto iterator = addresses.find(user.value);
    if( iterator == addresses.end() )
    {
      addresses.emplace(user, [&]( auto& row ) {
       row.key = user;
       row.first_name = first_name;
       row.last_name = last_name;
       row.street = street;
       row.city = city;
       row.state = state;
      });
      send_summary(user, "successfully emplaced record to addressbook");
    }
    else {
      std::string changes;
      addresses.modify(iterator, user, [&]( auto& row ) {
        row.key = user;
        row.first_name = first_name;
        row.last_name = last_name;
        row.street = street;
        row.city = city;
        row.state = state;
      });
      send_summary(user, "successfully modified record to addressbook");
    }
  }

  [[eosio::action]]
  void erase(name user) {
    require_auth(user);
    address_index addresses(_self, _code.value);
    auto iterator = addresses.find(user.value);
    eosio_assert(iterator != addresses.end(), "Record does not exist");
    addresses.erase(iterator);
    send_summary(user, "successfully erased record from addressbook");
  }

  [[eosio::action]]
  void notify(name user, std::string msg) {
    require_auth(get_self());
    require_recipient(user);
  }

private:
  
  struct [[eosio::table]] person {
    name key;
    std::string first_name;
    std::string last_name;
    std::string street;
    std::string city;
    std::string state;
    uint64_t primary_key() const { return key.value; }
  };

  void send_summary(name user, std::string message) {
    action(
      permission_level{get_self(),"active"_n},
      get_self(),
      "notify"_n,
      std::make_tuple(user, name{user}.to_string() + message)
    ).send();
  };

  typedef eosio::multi_index<"people"_n, person> address_index;

};

EOSIO_DISPATCH( addressbook, (upsert)(notify)(erase) )   

~~~



~~~
cleos set contract addressbook /contracts/addressbook
cleos push action addressbook upsert '["alice", "alice", "liddell", "123 drink me way", "wonderland", "amsterdam"]' -p alice@active
cleos get actions alice
~~~


## 2.6 Inline Action to External Contract

The only new concept in the code above, is that we are explicitly restricting calls to the one action to a specific account in this contract using require_auth to the addressbook contract, as seen below.

~~~

//Only the addressbook account/contract can authorize this command. 
require_auth( name("addressbook"));
//Only the addressbook account/contract can authorize this command. 
require_auth( name("addressbook"));
Previously, a dynamic value was used with require_auth


cleos create account eosio abcounter EOS6L6hTF9N83aekjpdfpkvBd19PhH2hhJjM8gYtouTM8fT8Q2oBA

eosio-cpp -o abcounter.wasm abcounter.cpp --abigen

cleos set contract abcounter /contracts/abcounter

cleos push action addressbook upsert '["alice", "alice", "liddell", "123 drink me way", "wonderland", "amsterdam"]' -p alice@active

cleos get table abcounter abcounter counts -k alice

cleos push action addressbook upsert '["alice", "alice", "liddell", "1 there we go", "wonderland", "amsterdam"]' -p alice@active

cleos push action addressbook erase '["alice"]' -p alice@active

cleos push action abcounter count '["alice", "erase"]' -p alice@active

cleos get table abcounter abcounter counts -k alice
~~~




