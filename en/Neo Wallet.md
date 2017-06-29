# Neo Wallet

### Introduce

Imagine a physical wallet in our pocket, there are several debit cards in it, eaching containing a balance. When we need to pay a bill, we will use these cards to pay it off. If the bill is high enough then one card might not have enough balance on it to pay off the entire bill. So we would use multiple cards in order to pay off the bill. The Neo wallet is the same as the wallet in our pocket, however, instead of cards, the Neo wallet uses addresses to hold balances.

![Comparison](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Comparison.jpg)



### Balance

##### Phyiscal Wallet Analogy
Imagine a scenario where I have two cards (or wallet ids), each containing a balance of ¥100. If I have a ¥100 bill to pay, then I can pay it off using only the first card. But if I have a ¥150 bill to pay, my first card will not be enough. I have to use both of my cards in order to pay off my bill. I use the full amount on both cards, and the remainder or change will be given back to me on another card of my choosing. Thus at the end of this transaction I own a card which has ¥50 worth of balance. 

This same scenario plays in the NEO wallet. I will have two wallet ID's instead of cards, and my change is returned back to me to a third wallet ID.

![Balance](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Balance.jpg)

##### Neo Wallet

If I want to transfer my coins, Neo wallet will calculate the total coins that I need to transfer. Then gather coins fromn the different addresses that are in my possesion to complete my transfer. The remainder will be given back to me as change. By default Neo will choose the first address as the default address when returning the change.

### Transfer

![Transfer](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Transfer.jpg)

For example, my wallet has 150 Neo Shares and 110 Neo Coins, I want to transfer 120 Neo Shares to my friend's address, buy my total coins are in 3 different addresses. The Neo wallet will calculate and make an optimal transfer strategy which is 100 Neo Shares from address1 and 20Neo Shares from address3. Then a transactuin will be constructed by tge wallet according to this strategy and send it to the neo block chain through tge p2p network protocol. The client wallet will  update the local balance in the mean time.

![Neo Wallet](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Neo%20Wallet.jpg)

If this tx is verified by different neo nodes, it will be recorded in one block, which means this tx is read to store in the blockchain. The wakllet will be noticed by the blockchain by its sync mechanism and confirm this tx. 

However, if this tx cannot pass the verification of neo nodes for some reason, it will not be confirmed by wallet. You will have to rebuild wallet index in order to erase this failed tx and make displayed balance match again.

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