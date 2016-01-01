Feel free to skip to [Step 2](https://github.com/gavingmiller/level-6/blob/gh-pages/explanation.md#step-2) which contains evidence explanation. First part is the lead in to evidence gathering.

Steps to Solve (play-by-play)
==============

* Can get the highest order id with:
  * `GET https://api.stockfighter.io/ob/api/venues/:venue/stocks/:stock/orders/99999999`
  * Returns order id of the highest order in the error response

* Can get an account number by canceling an order you don't own on:
  * `DELETE https://api.stockfighter.io/ob/api/venues/:venue/stocks/:stock/orders/:order`
  * Returns account number of originating account

Lo & Behold, an unauthorized cancel returns the account associated with the order.
Thus we're able to enumerate/discover the accounts that exist. Pseudocode:

```ruby
0.upto(highest_order_id) do |order_id|
  an = get_account_number_for_order_id(order_id)
  if !seen_an?
    accounts << an
  end
end
```

All of the strict API calls require authentication - as I found earlier searching for a timing attack ;) -
so we'll have to use the websockets to watch the fills. Quotes ought to be the same accross all of the accounts
so no need to monitor those.

### Step 1:
  1. Iterate accross X order_ids and "cancel" to discover all the bots that exist
  2. Create a new websocket to listen to all of the bots
  3. ...
  4. Profit!

In a dry run using the enumeration technique I was able to discover some other accounts:

```
YAS64459913, MM63117084, BY87868375, LKH6571236, ..., LYC73209006
```

It looks like we're getting 100 accounts per level instance. After 4000 cancels it seems fair to settle on that number.
That's substantially more than I assumed were running on a level o_O (will have to try account enumeration on other levels)

### Step 2:
  1. Create execution/fill web sockets foreach account
  2. Modify the fills to dump the original orders to log
  3. Wait 15 or so minutes
  3. Graph the results

Upon graphing the fills for all accounts, it's abundantly clear which account has made a substantial sum of money buying low and selling high,
and is exhibiting anomalous activity. The other actors present in the exchange are a combination of market makers, and the occasional
isolated purchase accounts.

Graphs can be viewed here: [http://gavingmiller.github.io/level-6/](http://gavingmiller.github.io/level-6/)

One of the tip off clues to the abnormal behavior of this particular account is that the sells generally remain above the buys.
This gives a visual tip that someone is using some measure of exactness when trading.

The final bit of information is the trade volume is more sparse than a market maker which has a much higher volume. Again indicating
a much different buy/sell strategy.

To date, account BFB55463548 made a profit of $44,041.00 by:

* Selling 25,797 shares for $2,171,398.00
* Buying 27,471 shares for -$2,298,022.00
* With 1674 shares outstanding @ $101.00 ($170,664.00).

While not definitive conviction worthy proof, this provides enough information to subpoena this account as being anomalous.
