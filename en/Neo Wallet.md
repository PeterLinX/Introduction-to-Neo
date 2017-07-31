# Neo Wallet

### Introduction

Imagine a physical wallet in our pocket, there are several debit cards in it, each containing a different balance. When we need to pay a bill, we will use these cards to pay it off. If the bill is expensive enough, one card alone might not have enough balance on it to pay off the entire bill. So we would use multiple cards to pay off the bill. The Neo wallet is the same as the wallet in our pocket, however, instead of cards, the Neo wallet uses addresses to hold balances.

![Comparison](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Comparison.jpg)



### Balance

##### Phyiscal Wallet Analogy
Imagine a scenario where I have two cards, each containing a balance of ¥100. If I have a ¥100 bill to pay, then I can pay it off using only the first card. But if I have a ¥150 bill to pay, my first card will not be enough. I have to use both of my cards in order to pay off my bill. I have to use the full amount on both cards, and the remainder or change will be given back to me on another card of my choosing. Thus at the end of this transaction I own a card which has ¥50 worth of balance. 

![Balance](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Balance.jpg)

##### Neo Wallet

If I want to transfer my coins, Neo wallet will calculate the optimal input addresses and the corresponding amounts to make up the total sum of the coins that I need to transfer. The transaction will be constructed from the full sum of coins from  each of the addresses being used for the transfer. After deducting the desired amount (which I input to transfer) to be sent from the full sum of coins, the remainder will be given back to me as change. By default Neo will choose the FIRST address of your wallet to be the default address to send the change coins.

### Transfer

![Transfer](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Transfer.jpg)

For example, my wallet has 150 Neo and 110 Neo Gas, I want to transfer 120 Neo to my friend's address, but my 150 Neo are stored in 3 different addresses. The Neo wallet will calculate and make an optimal transfer strategy which is 100 Neo from address1 and 20 Neo from address3. Then a transaction will be constructed by the wallet according to this strategy and send it to the neo blockchain through the p2p network protocol. The client wallet will  update the local balance in the mean time.

![Neo Wallet](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Neo%20Wallet.jpg)

If this tx is verified by multiple different neo nodes, it will be recorded in the next block on the blockchain, which means that the tx has been stored and recorded in the blockchain. The wallet will be notified by the blockchain by its sync mechanism and confirm this tx. 

However, if this tx does not pass the verification of neo nodes for some reason, it will not be confirmed by wallet and will not be recorded in the blockchain. But since your own wallet has updated your balance to reflect the transfer, you will have to rebuild wallet index in order to remove this failed tx and rescan the blockchain to restore the balance of Neo in each of the addresses in your wallet. Since the transaction has not been recorded on the blockchain, your Neo holdings will not have changed and you still "own" them. The best way to check is to go on a blockchain explorer and search for your address. Any unconfirmed tx will not show up in the blockchain as it has not be recorded in the blocks yet.

If you're curious about neo wallet's transfer strategy, please check our wallet code, and read the appendix blow. Neo block chain and wallet program is open source. You can build your own wallet by compiling the code hosted on github on your local machine. Enjoy!

### Appendix: Transfer Strategy

```c#
   public T MakeTransaction<T>(T tx, UInt160 change_address = null, Fixed8 fee = default(Fixed8)) where T : Transaction
   {
        if (tx.Outputs == null) tx.Outputs = new TransactionOutput[0];
        if (tx.Attributes == null) tx.Attributes = new TransactionAttribute[0];
        fee += tx.SystemFee;
        var pay_total = (typeof(T) == typeof(IssueTransaction) ? new TransactionOutput[0] : tx.Outputs).GroupBy(p => p.AssetId, (k, g) => new
        {
            AssetId = k,
            Value = g.Sum(p => p.Value)
        }).ToDictionary(p => p.AssetId);
        if (fee > Fixed8.Zero)
        {
            if (pay_total.ContainsKey(Blockchain.SystemCoin.Hash))
            {
                pay_total[Blockchain.SystemCoin.Hash] = new
                {
                    AssetId = Blockchain.SystemCoin.Hash,
                    Value = pay_total[Blockchain.SystemCoin.Hash].Value + fee
                };
            }
            else
            {
                pay_total.Add(Blockchain.SystemCoin.Hash, new
                {
                    AssetId = Blockchain.SystemCoin.Hash,
                    Value = fee
                });
            }
        }
        var pay_coins = pay_total.Select(p => new
        {
            AssetId = p.Key,
            Unspents = FindUnspentCoins(p.Key, p.Value.Value)
        }).ToDictionary(p => p.AssetId);
        if (pay_coins.Any(p => p.Value.Unspents == null)) return null;
        var input_sum = pay_coins.Values.ToDictionary(p => p.AssetId, p => new
        {
            AssetId = p.AssetId,
            Value = p.Unspents.Sum(q => q.Output.Value)
        });
        if (change_address == null) change_address = GetChangeAddress();
        List<TransactionOutput> outputs_new = new List<TransactionOutput>(tx.Outputs);
        foreach (UInt256 asset_id in input_sum.Keys)
        {
            if (input_sum[asset_id].Value > pay_total[asset_id].Value)
            {
                outputs_new.Add(new TransactionOutput
                {
                    AssetId = asset_id,
                    Value = input_sum[asset_id].Value - pay_total[asset_id].Value,
                    ScriptHash = change_address
                });
            }
        }
        tx.Inputs = pay_coins.Values.SelectMany(p => p.Unspents).Select(p => p.Reference).ToArray();
        tx.Outputs = outputs_new.ToArray();
        return tx;
    }
```



Edited by Peter Lin (https://github.com/PeterLinX/Introduction-to-Neo)
