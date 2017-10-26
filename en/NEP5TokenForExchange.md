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