# kh_data_modelling

The Data Model

The data model consists of three subject areas:

[] Currencies
[] Items
[] Traders

The Currencies subject area is simple. It contains four tables that store every currency we use and their exchange rates. Currencies are important because:

We will use one currency, called the base currency, for trading. An online stock trading platform will likely use the U.S. dollar (USD) as its base currency, regardless of the traders’ actual regions. All transactions will be converted into the base currency.
We can also have non-base or local currencies for all the countries where our trading platform is available. This would allow us to display prices in the local currency but still perform trades in the base currency.
The remaining two tables relate currencies and countries.

The most important table in this subject area is the currency table. This is where we’ll store all currencies we’ve ever used for trading, including cryptocurrencies. Whether a currency is included in this table depends on if that currency will be used to pay for the traded items. For each currency, we’ll store:

code – A code used to UNIQUELY denote that currency. For national currencies, this will be the ISO 4217 code (e.g. USD for United States Dollar) or some other official code. We could also use ISO 4217 for cryptocurrencies; XBT is Bitcoin’s ISO code. However, Bitcoin also uses the code BTC informally.
name – That currency’s UNIQUE name (e.g. United States Dollar).
is_active – If the currency is currently active in our system.
is_base – If this currency is our system’s base currency. Usually, we’ll have only one base currency at a time. It’s possible we could have more than one, such as using euros for EU states and US dollars for other areas. In that case, we have the capability assign a base currency to each country with this attribute.
The next table stores current and historical rates between currency pairs. In the currency_rate table, we’ll store the currency_id we want to compare to a base_currency_id as well as the rate when this pair was stored (ts). Since we’ll store rates as they were at various points in time, this table will store both historical and current data.

A list of all relevant countries is stored in the country dictionary. Besides the primary key (id), it contains one attribute that holds a UNIQUE country name.

The last table in this subject area is the currency_used table. In most cases, a country will always use the same currency. Still, changes can happen, like when many EU countries replaced their national currencies with the euro. To cover such an eventuality, we’ll store a history of all the currencies we’ve used. For each record in this table, we’ll store references to the country table (country_id), the currency table (currency_id), and when this currency was used (date_from and date_to). If date_to is NULL, then this currency is currently in use. Of course, only one currency should be in use per country. We won’t implement that check in the model; instead, we’ll perform a check when a record is added or updated in this table.



Tables in the Items subject area define all items available for trade and their current status. It also records all changes that happened to these items over time.

The item table lists all items that traders can buy or sell (or that they have bought or sold). These could be stocks, funds, or cryptocurrencies. Any trade involving these financial instruments uses almost exactly the same process, so we can use the same structure for all of them. For each item, we’ll store:

code – A UNIQUE text code for that item, similar to what we use for shares (e.g. NASDAQ uses the code “AAPL” for Apple Inc).
name – The full name of the company (for shares), fund, or cryptocurrency.
is_active – Whether this item is available for trade or not.
currency_id – References the currency used as base currency for this item.
details – All additional details (such as the number of shares issued) in textual format.
The price table tracks all price changes across time. Once a change has occurred, we’ll store the time (ts), and the buy and sell price for the item (item_id) involved. We’ll also store a reference to the currency table, which tells us the currency used to set the value of that item at that time. Notice that preferred currency for any item could change.

The final table in this subject area is the report table. The idea is to store regular (i.e. daily) reports for each item. This report will be based on trading during that period, and it will keep financial details in one place. This is redundant data, but it can prove to be very useful when querying historical prices (which happens often, as traders are extremely interested in trends). For each record in this table, we’ll store:

trading_date – The date of this report. If we need to compile reports more often than once a day, we’ll have to make changes to the model – e.g. adding timestamps that indicate when a trading period began and ended.
item_id and currency_id – References the related item and the currency used.
first_price, last_price, min_price, max_price and avg_price – The first, last, maximum, minimum, and average price for this item during this period.
total_amount – The total amount paid for that item during the reporting period.
quantity – The number (quantity) of items traded during this reporting period. Please note that an average price could be calculated from total_amount and quantity, but I prefer to keep “total_amount” separate. This simplifies the situation when we create a report for a longer time period, such as weekly. In that case, we could add all the total_amount attributes and divide them by the sum of all quantity attributes to get a weekly average price.
All attributes in this table (other than the primary key and the foreign keys) can be NULL. This will be the case when we insert a record for a new trading period – there are no trades so far. At the beginning of each date, we can expect we’ll insert one record for every item and update these values as the day progresses. The final updated value will also be the final report for that day.

The Traders subject area is the last one we’ll discuss, but it’s the most important area in the model. Its four tables (leaving out the country and item tables that we’ve already covered) store information about traders, their inventories, and their actions. Note that the currency table used here is just a copy. It’s used to simplify the model and avoid relations overlapping.

The central table is the trader table. For each trader, we’ll store:

first_name and last_name – The trader’s first and last names.
user_name and password – The username and password (hash) chosen by the trader. The user_name attribute can store only UNIQUE values.
email – The trader’s email address. This will be used to complete the registration process and for all subsequent contacts with the trader. It can also hold only UNIQUE values.
confirmation_code – The code sent to the user to complete the registration process.
time_registered and time_confirmed – Timestamps of when the trader registered and when they completed the registration process.
country_id – The country where the trader lives.
preferred_currency_id – The currency that the trader wants prices displayed in.
The list of all items a trader currently owns is stored in the current_inventory table. For each UNIQUE trader_id – item_id pair, we’ll store the quantity the trader currently owns.

The remaining two tables are directly related to offers and trades. We’ll assume that each trader can place an offer to buy or sell items at a certain price. When a matching offer appears, the trade event will happen. (We won’t go into details that are specific to stock exchanges, where a broker serves as a mediator.)

We’ll keep a record of all offers in the offer table. Any trader can place an offer to buy or sell items. To make this happen, we need to store the following details:

trader_id and item_id – References the trader who placed that offer and the item they want to buy or sell.
quantity – The quantity they want to buy or sell.
buy and sell – If this offer is for buying or selling. Only one attribute can be set at a time.
price – The desired buying or selling price. It’s not required because a trader may want to buy or sell no matter what the price is.
ts – The timestamp when this record was inserted.
is_active – Whether this offer is still active. It could become inactive a) if the trader sets it to inactive, or b) if the trade has taken place.
The final table in our model contains data related to the trading event. Trading takes place between two users after they both place an offer. The price used could be the first price offered or the current price, depending on what we want to implement in our application. For each trade event, we’ll store:

item_id – Refers to the item traded.
seller_id and buyer_id – Both reference the trader table and denote the users involved in the trade.
quantity – How much of that item was traded in this transaction.
unit_price – The unit price used for this item in this trade.
description – All additional details, in textual format.
offer_id – The ID of the offer that initiated this trade. Note: The first offer initiates a trade, so that’s the ID we’ll store here.
ts – The timestamp when this trade happened.

