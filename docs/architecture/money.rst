Handling Money Amounts
======================

Saleor uses `Prices <https://github.com/mirumee/prices/>`_ together with `django-prices <https://github.com/mirumee/django-prices/>`_ to store, calculate and display amounts of money, prices and ranges of those.

Those libraries come with certain design decisions that reflect decisions made when developing Saleor and different from other popular libraries:

Default currency
----------------

By design all amounts and prices are entered and stored in single currency, specified in the `DEFAULT_CURRENCY` setting. Saleor is capable of displaying prices in user's currency, but scenario where store offers different items with prices in different currencies is not supported. Additionally, no conversion is performed on amounts used to calculate orders total prices or sent to payment providers.

This also means that changing currency in active store while possible, has unpredictable results and is not supported.

Money and TaxedMoney
--------------------

`Money` is a type representing amount of money in specific currency, eg. `Money(100, currency='USD')` means 100 USD. This type doesn't hold any additional information useful for commerce, but unlike `Decimal` it implements safeguards and checks important for calculations and comparisions different monetary values and will raise errors if you are trying to do something bad like adding dollars to euros. Money amount is stored on model using `MoneyField` that provides its own safechecks on currency and precision of stored data. If you ever need get to the `Decimal` of your `Money` object, you'll find it on the `amount` property.

Products and shipping methods prices are stored using `MoneyField` and are assumed to be gross amounts. Those amounts are then naively converted to `TaxedMoney` objects using the `TaxedAmount(net=item.price, gross=item.price)` approach, usually together with discount computation.

.. note::
  
  In future Saleor will support setting for store administrators to change this approach to use net values instead if this fits their business more.

TaxedMoneyRange
---------------

Sometimes a product may be available under more than single price due to it's variants defining custom prices different from base brice. For such situations `Product` defines additional `get_price_range` and `get_gross_price_range` methods that return `TaxedMoneyRange` object defining minim and maximum prices on its `start` and `stop` attributes. This object is then used by the UI to differentatie between displaying price as "10 USD" or "from 10 USD" in case of products where prices differ between variants.

TaxedMoneyRange is also used in for displaying other money ranges, like min and maximum cost and margin on product's variant in dashboard.