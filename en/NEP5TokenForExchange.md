### 1.Deposit

#### 1.1 Run neo-cli
>dotnet neo-cli.dll --rpc --record-notifications

#### 1.2 Receive Notifications
If you need to get the notification of user's deposit, you can simply add parameter "--record-notifications". A document file called "notifications" will be generated in root path. 
![img](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/NEP5TokenForExchange/1.jpg)

Notifications in every block will be recorded in a json file such as: 
![img](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/NEP5TokenForExchange/2.jpg)

#### 1.3 Notifications json file
For example, the content of the file "block-1503847" is: 
```json
[
{
    "txid": "0x65d62a736a73c4d15dc4e4d0bfc1e4bbc4ef220e163625d770eb05577b1afdee",
    "contract": "0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9",
    "state":
    {
        "type": "Array",
        "value": [
        {
            "type": "ByteArray",
            "value": "7472616e73666572"
        },
        {
            "type": "ByteArray",
            "value": "d336d7eb9975a29b2404fdb28185e277a4b299bc"
        },
        {
            "type": "ByteArray",
            "value": "eab336cac807707295afa7e7da2f4683237f612a"
        },
        {
            "type": "ByteArray",
            "value": "006ad42d100100"
        }]
    }
}]
```
There is an array of notifications in this json file. There is only one object in the array, which means only one NEP5 event triggered in the block 1503847.

#### 1.4 Analyse json file

(1) Filter contract by checking the value of "contract". Then you can have the right asset type.  
(2) Filter the function by checking the first object in "state". If the event is "transfer" then the value of the "state" will be:
- An array of 4 objects: [event, from, to, amount]. It includes all information of a transaction;
- The first object of the array will be the name of the "transfer" event : 
```json
{
	"type": "ByteArray",
	"value": "7472616e73666572"
}
```
(3) Filter the third object in "state". If it is the address of the exchange then you get a notification of deposit.

### 2.Query

If you need to query someone's balance you should invoke 3 functions in NEP5 which are "balanceOf", "decimals" and "symbol". Then You can get the correct balance.

#### 2.1 Use RPC API "invokefunction"
You can use rpc api "invokefunction" by sending json to neo rpc server to query someone's balance. There're 3 parameters that you need to set up.

(1) Script hash  
Set up the script hash of the NEP5 token you are querying. For example, you can find the script hash of RPX is : 0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9.

(2) The name of the method  
Set up the name of the method you are invoking. According to NEP5，if someone need to query his token balance he should invoke function "balanceOf".

(3) The arguments of the method  
According to NEP5，there're two parameters in the main function. The first parameter is the name of methond that you need to invoke. The second parameter is an array, which is optional. If the method you are invoking need some arguments, you can passing them by constructing these parameters into an array. For example, "balanceOf" methond in NEP5 returns the token balance of the '''account'''.  
<code>public static BigInteger balanceOf(byte[] account)</code>  
So you need to pass the account info as an argument in the "balanceOf" method.

#### 2.2 Invoke "balanceOf" function

If the address of the account is AJShjraX4iMJjwVt8WYYzZyGvDMxw6Xfbe, you need to convert it into Hash160 type and construct this parameter as a json object such as:
```json
{
    "type": "Hash160",
    "value": "bfc469dd56932409677278f6b7422f3e1f34481d"
}
```
Then you can construct the whole json message such as:

Request Body：

```json
{
  "jsonrpc": "2.0",
  "method": "invokefunction",
  "params": [
    "ecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9",
    "balanceOf",
    [
      {
        "type": "Hash160",
        "value": "bfc469dd56932409677278f6b7422f3e1f34481d"
      }
    ]
  ],
  "id": 3
}
```

After sending the request, you will get the following response：

```json
{
    "jsonrpc": "2.0",
    "id": 3,
    "result": {
        "state": "HALT, BREAK",
        "gas_consumed": "0.338",
        "stack": [
            {
                "type": "ByteArray",
                "value": "00e1f505"
            }
        ]
    }
}
```
It returns "00e1f505" which can be converted to interger 100000000.

#### 2.3 Invoke "decimals" function

Request Body：

```json
{
  "jsonrpc": "2.0",
  "method": "invokefunction",
  "params": [
    "0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9",
    "decimals", 
    []
    ],
  "id": 2
}
```

After sending the request, you will get the following response：

```json
{
    "jsonrpc": "2.0",
    "id": 2,
    "result": {
        "state": "HALT, BREAK",
        "gas_consumed": "0.156",
        "stack": [
            {
                "type": "Integer",
                "value": "8"
            }
        ]
    }
}
```
It returns integer 8.

#### 2.4 Invoke "symbol" function

Request Body：

```json
{
  "jsonrpc": "2.0",
  "method": "invokefunction",
  "params": [
    "0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9",
    "symbol", 
    []
    ],
  "id": 1
}
```

After sending the request, you will get the following response：

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "state": "HALT, BREAK",
        "gas_consumed": "0.141",
        "stack": [
            {
                "type": "ByteArray",
                "value": "525058"
            }
        ]
    }
}
```
It returns "525058" which can be converted to string "RPX".

#### 2.5 Calculate the correct balance
According these three important infos by invoking functions in NEP5, we can get the correct balance.
The balance = 100000000/10^8 RPX = 1 RPX


### 3.Withdraw

There're 3 ways for exchange to send asset to users.  
(1)neo-cli command: send  
(2)rpc method: sendtoaddress  
(3)rpc method: sendmany  

#### 3.1 neo-cli command: send

`send <txid|script hash> <address> <value> [fee = 0]`

There're 4 parameters. The first parameter is the asset ID, the second parameter is the payment address, the third parameter is the transfer amount, and the fourth parameter is the fee. (This parameter can be left empty, and the default is 0) The command needs to verify the wallet password. For example, in order to transfer 100 RPX to the address "AeSHyuirtXbfZbFik6SiBW2BEj7GK3N62b", you need to enter the following command.

`send 0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9 AeSHyuirtXbfZbFik6SiBW2BEj7GK3N62b 100`

If you need to send global asset, just change the first parameter to txid. For example,  
The txid of NEO: 0Xc56f33fc6ecfcd0c225c4ab356fee59390af8560be0e930faebe74a6daff7c9b  
The txid of GAS: 0x602c79718b16e442de58778e148d0b1084e3b2dffd5de6b7b16cee7969282de7  

#### 3.2 rpc method: sendtoaddress

The value of key "params" is an array of 4 parameters. 

`"params":[script hash, address, amount, fee(optional), change address(optional)]`

For example, if I send 1 RPX to AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg , then I can construct a json below and send it to rpc server.

Request Body：

```json
{
    "jsonrpc":"2.0",
    "method":"sendtoaddress",
    "params":[
        "0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9",
        "AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg",
        "1",
        "0",
        "ARkJ8QcVdYL68WRvN3wj3TSvXX8CgmC73Z"
    ],
    "id":1
}
```

After sending the request, you will get the following response：

```json
{
    "jsonrpc":"2.0",
    "id":1,
    "result":{
        "txid":"0xc6d4bf7c62fb47e0b2a6e838c3a1ca297622a1b1df7ceb2d30fa4ef8b7870700",
        "size":219,
        "type":"InvocationTransaction",
        "version":1,
        "attributes":[
            {
                "usage":"Script",
                "data":"5305fbbd4bd5a5e3e859b452b7897157eb20144f"
            }
        ],
        "vin":[

        ],
        "vout":[

        ],
        "sys_fee":"0",
        "net_fee":"0",
        "scripts":[
            {
                "invocation":"4054fbfca678737ae164ebf0e476da0c8215782bc42b67ae08cf4d8a716eeef81fcc17641e7f63893c3e685fb7eb1fb8516161c5257af41630f4508dde3afa3a8c",
                "verification":"210331d1feacd79b53aeeeeb9de56018eadcd07948675a50258f9e64a1204b5d58d1ac"
            }
        ],
        "script":"0400e1f50514d710f6f3f0bad2996a09a56d454cfc116a881bfd145305fbbd4bd5a5e3e859b452b7897157eb20144f53c1087472616e7366657267f91d6b7085db7c5aaf09f19eeec1ca3c0db2c6ecf166187b7883718089c8",
        "gas":"0"
    }
}
```

#### 3.3 rpc method: sendmany
The value of key "params" is an array of 4 parameters. 

`"params":[[], fee(optional), change address(optional)]`

For example, if I send 15.5 RPX and 0.0001 GAS to AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg and the change address is also set to AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg, then I can construct a json below and send it to rpc server.

Request Body：

```json
{
    "jsonrpc":"2.0",
    "method":"sendmany",
    "params":[
        [
            {
                "asset":"0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9",
                "value":"15.5",
                "address":"AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg"
            },
            {
                "asset":"0x602c79718b16e442de58778e148d0b1084e3b2dffd5de6b7b16cee7969282de7",
                "value":"0.0001",
                "address":"AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg"
            }
        ],"0.00001","AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg"
    ],
    "id":1
}
```

After sending the request, you will get the following response：

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "txid": "0xe1351c9c9f2205a801d1b04f0df2d65fb4b1692d7d3b06cf41e0712fd1b12c9c",
        "size": 373,
        "type": "InvocationTransaction",
        "version": 1,
        "attributes": [
            {
                "usage": "Script",
                "data": "6d64dc9e50af8e911247436b264c8f7d791ad58c"
            }
        ],
        "vin": [
            {
                "txid": "0x9f0a28a912527604ab4b7d5e8b8d1a9b57631fcbab460132811ae7b6ed1ccaff",
                "vout": 1
            }
        ],
        "vout": [
            {
                "n": 0,
                "asset": "0x602c79718b16e442de58778e148d0b1084e3b2dffd5de6b7b16cee7969282de7",
                "value": "0.0001",
                "address": "AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg"
            },
            {
                "n": 1,
                "asset": "0x602c79718b16e442de58778e148d0b1084e3b2dffd5de6b7b16cee7969282de7",
                "value": "0.01359",
                "address": "AbP3FU3YcqBrWh72nc9deyQB99eazG9XUg"
            }
        ],
        "sys_fee": "0",
        "net_fee": "0.00001",
        "scripts": [
            {
                "invocation": "40644ab915419dbf855a52d5c75596e80b78c8e928cc0ce91ae6afc3b75a0c31ee54efe1836f9ec232f6c42dcb3ace0bfdc688e626944fa20970a76064975eade9",
                "verification": "2103d4b6fc2d116855f86a483d151182f68e88e6ddd13f3f1f3631e36300aac122bfac"
            }
        ],
        "script": "04801f635c14d710f6f3f0bad2996a09a56d454cfc116a881bfd146d64dc9e50af8e911247436b264c8f7d791ad58c53c1087472616e7366657267f91d6b7085db7c5aaf09f19eeec1ca3c0db2c6ecf166f871fb30fc859b77",
        "gas": "0"
    }
}
```


### Appendix: NEP5 Token Standard

You can find the original description here: [Token Standard](https://github.com/neo-project/proposals/blob/master/nep-5.mediawiki "NEP5")

In the method definitions below, we provide both the definitions of the functions as they are defined in the contract as well as the invoke parameters.

This standard defines two method types:

* '''(Required)''' : methods that are present on all NEP5 tokens.

* '''(Optional)''' : methods that are optionally implemented on NEP5 tokens. These method types are not required for standard interfacing, and most tokens should not use them. All optional methods must be enabled if choosing to use them.

#### Methods

##### totalSupply

* Syntax: <code>public static BigInteger totalSupply()</code>

* Remarks: "totalSupply" returns the total token supply deployed in the system.

##### name

* Syntax: <code>public static string name()</code>

* Remarks: "name" returns the token name.


##### symbol

* Syntax: <code>public static string symbol()</code>

* Remarks: "symbol" returns the token symbol.

##### decimals

* Syntax: <code>public static byte decimals()</code>

* Remarks: "decimals" returns the number of decimals used by the token.

##### balanceOf

* Syntax: <code>public static BigInteger balanceOf(byte[] account)</code>

* Remarks: "balanceOf" returns the token balance of the '''account'''.

##### transfer

* Syntax: <code>public static bool transfer(byte[] from, byte[] to, BigInteger amount)</code>

* Remarks: "transfer" will transfer an '''amount''' of tokens from the '''from''' account to the '''to''' account.

##### allowance ''(optional)''

* Syntax: <code>public static BigInteger allowance(byte[] from, byte[] to)</code>

* Remarks: "allowance" will return the amount of tokens that the '''to''' account can transfer from the '''from''' acount.

##### transferFrom ''(optional)''

* Syntax: <code>public static bool transferFrom(byte[] originator, byte[] from, byte[] to, BigInteger amount)</code>

* Remarks: "transferFrom" will transfer an '''amount''' from the '''from''' account to the '''to''' acount if the '''originator''' has been approved to transfer the requested '''amount'''.

##### approve ''(optional)''

* Syntax: <code>public static bool approve(byte[] originator, byte[] to, BigInteger amount)</code>

* Remarks: "approve" will approve the '''to''' account to transfer '''amount''' tokens from the '''originator''' acount. 

#### Events

##### transfer

* Syntax: <code>public static event Action<byte[], byte[], BigInteger> transfer</code>

* Remarks: The "transfer" event is raised after a successful execution of the "transfer" method.

#### Implementation

*Woolong: https://github.com/lllwvlvwlll/Woolong  
*ICO Template: https://github.com/neo-project/examples/tree/master/ICO_Template