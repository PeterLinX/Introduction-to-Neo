### 1.Deposit

#### 1.1 Run neo-cli
>dotnet neo-cli.dll /rpc --record-notifications

#### 1.2 neo-cli will generate a document file which have all notifications in its root path.
![img](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/NEP5TokenForExchange/1.jpg)

#### 1.3 You can analyst these json file
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
1.3.1 Filter contract by checking "contract"

1.3.2 Filter the function by checking first parameter of "value"

1.3.3 If the function is "transfer" then the value of the key "value" will be an array [transfer, from, to, amount]. So you can get all information of a transaction.

1.3.4 Filter the "to" info. If it is the address of the exchange then you get a notification of deposit.


### 2.Query

You can use rpc api "invokefunction" by sending json to neo rpc server to query someone's balance.

#### 2.1 Set up the script hash of the NEP5 token you are querying
For example, you can find the script hash of RPX is : 0xecc6b20d3ccac1ee9ef109af5a7cdb85706b1df9.

#### 2.2 Set up the name of method
According to NEP5，if someone need to query his token balance he should invoke function "balanceOf".

#### 2.3 Set up the arguments of the method
According to NEP5，there're two parameters in the main function. The first parameter is the name of methond that you need to invoke. The second parameter is an array, which is optional. If the method you are invoking need some arguments, you can passing them by constructing these parameters into an array.

#### 2.4 Construct the json message

For example, "balanceOf" methond in NEP5 returns the token balance of the '''account'''.
<code>public static BigInteger balanceOf(byte[] account)</code>
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
It returns the balance of the account.

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