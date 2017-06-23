# Neo Wallet

### Introduce

Imagine a wallet in our pocket, there're several cards in it. When we want to pay the bill, we will calculate the money we have first. Neo wallet is the same as the wallet in our pocket, however, it have addresses instead of cards.

![Comparison](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/images/Neo%20Wallet/Comparison.jpg)



### Balance

##### Wallet

For example, if I have a ¥100 bill to pay, I just use card 1 and pay it. But if I have a ¥150 bill to pay, card 1 will be not enough. I have to use card 1 and 2 to pay it together, then ¥50 as change will be given back to me and I choose a card to get that change.

![Balance](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/images/Neo%20Wallet/Balance.jpg)

##### Neo Wallet

If I want to transfer my coins, Neo wallet will calculate the total coins that you need to transfer, and use enough coins in different addresses to accomplish your transfer, then the change will be given back to you and neo wallet choose first address in your wallet to accept those change. 

### Transfer

![Transfer](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/images/Neo%20Wallet/Transfer.jpg)

For example, my wallet have 150ANS and 110ANC, I want to transfer 120ANS to my friend's address. My total coins are in 3 different address, wallet will calculate and make an optimal transfer strategy which is 100ANS from address1 and 20ANS from address3. Then wallet will construct a tx according to this strategy and send it to neo block chain through p2p network protocol, in the meantime, wallet's balance will make changes. 

![Neo Wallet](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/images/Neo%20Wallet/Neo%20Wallet.jpg)

If this tx is verified by different neo nodes, it will be record in one block, which means this tx is effective. Wallet will be noticed by block chain by its sync mechanism and confirm this tx. However, if this tx cannot pass the verification of neo nodes for some reason, it will not be confirmed by wallet. I have to rebuild wallet index to erase this failed tx and make balance right again.

Maybe you're curious about how did wallet get this transfer strategy, you can check our wallet code to find it out or just check appendix. Neo block chain and wallet program is open source. You can build your own wallet by compiling those codes in github by yourself. Enjoy!

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



Edited by Peter Lin



