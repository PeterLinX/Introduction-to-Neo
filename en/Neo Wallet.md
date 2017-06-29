# Neo Wallet

### Introduce

Imagine a wallet in our pocket, with several cards in it. Before we pay the bill, we would first have to calculate the amount of money in our wallet. Neo wallet works exactly like the wallet in our pocket, however instead of having cards, the cards are replaced with addresses.

![Comparison](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Comparison.jpg)



### Balance

##### Wallet

For example, when the bill is ¥100, I would simply use card 1 to pay the bill. However, if the bill ¥150, card 1 will not be sufficient to cover the bill, on its own. I would have to use card 1 and 2 together, where a change of ¥50 will be returned to me. (To the card of my choosing)

![Balance](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Balance.jpg)

##### Neo Wallet

The payment process for NEO wallet is similar. First, NEO wallet will calculate the total amount of coins that I want to transfer, and use enough coins from different addresses to make that transaction possible. If there is excess funds, the change will be returned to me. Note: Neo wallet sets the first address in the wallet, as the default change address, to receieve any excess change. 

### Transfer

![Transfer](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Transfer.jpg)

For example, I want to transfer 120 NEO to my friend's address. I have a wallet containing 150NEO and 110 NEO Coins, split up between three different addresses. As mentioned above, the wallet will calculate and decide on the optimal transfer strategy to execute this transaction, of 120 NEO to my friend. In this case, it would be 100 NEO from address1 and 20 NEO from address3. A tx will be constructed by the wallet and be broadcasted to the NEO blockchain, through the p2p NEO network protocol. In the meantime, my wallet's balance will be adjusted accordingly.

![Neo Wallet](https://github.com/PeterLinX/Introduction-to-Neo/blob/master/en/images/Neo%20Wallet/Neo%20Wallet.jpg)

If this tx is verified by different NEO nodes, and is recorded in one of the blocks, this means that the transacton is effective and valid. As a result, 120 NEO would be credited to my friend's NEO address, which will be confirmed when his wallet syncs and verifies the incoming transaction.

However, if for some reason, the transcation fails to pass the verification of the NEO nodes, it would not be recorded in any mined blocks. The transaction is invalid and my friend receives 0 NEO, even after updating his wallet to the latest block. This is because the 120 NEO are still in my wallet, due to the failed transaction. NOTE : In order to make my balance right again, I have to rebuild my wallet index and erase this failed transaction.

If you're curious about NEO's wallet transfer strategy, please check our wallet code to find out more, or refer to the appendix below. NEO blockchain and wallet programs are open source. You can build your own wallet by compiling these codes in github. Enjoy!

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



