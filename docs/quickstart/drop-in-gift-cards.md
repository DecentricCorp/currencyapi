# Drop-in Gift Cards
Lightrail's Drop-in Gift Card solution makes it easy to offer gift cards to your customers. 
Integrating into Lightrail is an easy process that can be completed in no more than a few days.

The solution is component based, using widgets which are created from simple HTML snippets.
When your customers receive a gift card, they can easily redeem and apply the gift card value to their account.
Your checkout process requires a simple update to accept payment from your customer accounts.  

## Getting Started
[Sign up](https://www.lightrail.com/app/#/register) for a Lightrail account. 

Note this quickstart assumes you are using Stripe to process payments: if you are using another payment processor and want to build a custom solution, please [contact us](mailto:hello@lightrail.com).

Configure your Drop-in Gift Card [template](https://www.lightrail.com/app/#/cards/dropin) within your Lightrail account to customize the appearance of widgets and gift card emails.
(For development, toggle your Lightrail account to test mode, this will allow you to use Stripe's test credit cards.) 

This is also where you'll connect your Stripe account and provide the URL to a redemption page where customers can redeem their gift cards (see Step 2).

If at any point you want to see a working example of the entire Drop-in Gift Card solution, check out our [sample app](https://github.com/Giftbit/stripe-integration-sample-webapp). 

## Step 1: Selling Gift Cards
The Gift Card Purchase Widget allows your customers to purchase gift cards from your site. 
Lightrail powers the entire gift card purchase and delivery flow. 
![Gift card purchase widget](assets/purchase-widget.png)

What you see here is our fictional brand called Rocketship. Once set up with our Drop-in solution, you will see your branding instead.

Add the following snippet to your gift card purchase page: 

```html
<div>
    <script 
        src="https://embed.lightrail.com/dropIn/cardPurchase.js"
        data-shoppertoken="{shopperToken}"> 
    </script>
    <!-- The Shopper Token acts as a public api token that is used for issuing the gift card. -->
    <!-- See below for details.  -->
</div>
```
The gift card is automatically delivered to the recipient in a branded email. The email includes a button to apply the gift card to the recipient's account.

## Step 2: Redeeming Gift Cards
The Gift Card Redemption Widget enables your customers to redeem gift cards to their account.
![Gift card purchase widget](assets/redemption-widget.png)

When the recipient clicks the "apply to account" button in the email, they are taken to the redemption page indicated in your [Drop-in template](https://www.lightrail.com/app/#/cards/template).

Add the following snippet to your redemption page:

```html
<div class="redemption-widget">
    <script
        src="https://embed.lightrail.com/dropIn/codeRedemption.js"
        data-shoppertoken="{shopperToken}"
        data-fullcode="{giftCode}">
    </script>
     <!-- The gift code must be passed into the widget. Ideally passed automatically from the url. -->
</div>
```
When a gift card is redeemed, the Redemption Widget applies the gift card amount to the customer's account. If the customer does not have an account in Lightrail already, a new account will be created automatically.

The Redemption Widget uses a Shopper Token generated by your system to identify the customer and automatically apply the gift card to their account (see [below](#shopper-tokens) for details about Shopper Tokens). Note, ensure your redemption page lives behind the login wall of your site so that the correct Shopper Token is generated by your system to identify the customer. 

Next, your existing checkout process needs to be modified to allow the customer to pay with their account, which now contains the balance of the gift card they received. 

## Step 3: Checkout

### Displaying Account Balance
To start, use the following snippet to display a customer's account balance:
```html
<span>
    <script
        src="https://embed.lightrail.com/dropIn/accountBalance.js"
        data-shoppertoken="{shopperToken}">    
    </script>
</span>
```
This gives your customer the information to choose whether or not to apply their account credit to their purchase. In our [sample webapp](https://github.com/Giftbit/stripe-integration-sample-webapp/blob/master/shared/views/checkout.html), the customer simply selects a checkbox to use their account credit.

### Accept Payment
You will need to add a custom script to your checkout page to apply your customer's account credit to their purchase and accept a secondary payment method (such as a credit card) to cover any remaining balance. 

This script will need to do the following: 

- Let the customer choose whether to use their account credit
- Adjust the split point accordingly: i.e. how much can be charged to their Lightrail account and how much to charge to their credit card
- Call a charge simulation method on your server (see next section)
- Post the charge (see next section)

For an example, we recommend that you take a look at the [checkout page of our sample webapp](https://github.com/Giftbit/stripe-integration-sample-webapp/blob/master/shared/views/checkout.html). The sample checkout handles the logic of splitting the transaction between the customer's account credit and Stripe; it also loads a Stripe Elements form to handle the credit card portion of the payment if needed. (Templating in the example is done with Mustache but is not required.)

### Post the Transaction (server side)
The transaction is handled by backend methods using one of our client libraries (or methods that you [write yourself](https://github.com/Giftbit/Lightrail-API-Docs/blob/drop-in-gift-cards/use-cases/stripe-split.md)). You'll need to set up two endpoints to handle submissions from the custom form you added to [accept payment](#accept-payment) to: 
1. simulate charges (check the customer's account balance),
1. post the charge and redirect your customer to a success page. 

#### Simulate a Charge / Balance Check
When your customer chooses to use their account credit, you need to see whether the account can cover the requested amount. In our [sample webapp](https://github.com/Giftbit/stripe-integration-sample-webapp/blob/master/shared/views/checkout.html), this is done by having the frontend call a `/simulate` endpoint that makes use of a method from one of our client libraries to simulate posting a charge to a Lightrail account. Here's a Node example:

```javascript
/**
 * REST endpoint that simulates the charge and returns JSON.
 */
function simulate(req, res) {
    const splitTenderParams = {
        userSuppliedId: uuid.v4(),
        nsf: false,
        shopperId: req.body.shopperId,
        currency: req.body.currency,
        amount: req.body.amount
    };

    // Try to charge the whole thing to lightrail, and we'll use the amount that would actually get
    // charged when we do the real transaction.
    const lightrailShare = splitTenderParams.amount;

    lightrailStripe.simulateSplitTenderCharge(splitTenderParams, lightrailShare)
        .then(transaction => {
            res.send(transaction.lightrailTransaction);
        })
        .catch(err => {
            // You'll want to actually handle any errors that come back
            console.error("Error simulating transaction", err);
            res.status(500).send("Internal error");
        });
}
```
Use the Lightrail transaction value that comes back to set the parameters for actually posting the charge. 

#### Post the Charge
Our client libraries make it easy to post a split-tender charge where part of the transaction is covered by a Lightrail account, and part is covered by Stripe. Here's a Node example of how to handle posting a charge: 

```javascript
/**
 * REST endpoint that performs the charge and returns HTML.
 */
function charge(req, res) {
    const splitTenderParams = {
        amount: req.body.orderTotal,      // From your cart/order object
        currency: req.body.currency,      // From your cart/order/store config
        source: req.body.source,          // Stripe payment 'source' or 'customer'
        shopperId: req.body.shopperId,    // Lightrail contact identifier; see below
        userSuppliedId: req.body.orderId       // Unique transaction identifier for idempotency
    };

    // Validate the amount to actually charge to Lightrail
    const lightrailShare = req.body.lightrailAmount;
    if (lightrailShare < 0) {
        res.status(400).send("Invalid value for Lightrail's share of the transaction");
    }

    // Use the Lightrail-Stripe integration library to create the split tender charge
    lightrailStripe.createSplitTenderCharge(splitTenderParams, lightrailShare, stripe)
        .then(splitTenderCharge => {
            // Redirect to your success page
            res.render("checkoutComplete.html", {
                lightrailTransactionValue: splitTenderCharge.lightrailTransaction ? splitTenderCharge.lightrailTransaction.value / -100 : 0,
                stripeChargeValue: splitTenderCharge.stripeCharge ? splitTenderCharge.stripeCharge.amount / 100 : 0
            });
        })
        .catch(err => {
            // You'll want to actually handle any errors that come back
            console.error("Error creating split tender transaction", err);
            res.status(500).send("Internal error");
        });
}
```

At this point, the charge has been posted to both Lightrail and Stripe. You can handle post-checkout flow as you otherwise would. 

## Authentication
Create your Lightrail API key from the [Integrations](https://www.lightrail.com/app/#/account/api) section of your Lightrail account.
Your Lightrail API key is used to complete the server side requests from checkout, and also to generate Shopper Tokens which are passed into the widgets.  

### Shopper Tokens
Shopper Tokens act like customer-specific API tokens for use in the drop-in widgets. 
They are based on a unique customer identifier from your e-commerce system: the `shopperId`. This is what links the customer from your system to their account in Lightrail. 

You must generate them server side using one of our [client libraries](https://github.com/Giftbit/Lightrail-API-Docs/blob/master/docs/client-libraries.md). (If you are working in a language that we don't currently offer a client library for, please [contact us](mailto:hello@lightrail.com) to discuss creating your own tokens.) 

You'll need an API key along with your shared secret key from the Integrations section of your account (see above).
For example, using the Lightrail Javascript client the Shopper Token can be created as follows:
```javascript
lightrail.configure({
    apiKey: process.env.LIGHTRAIL_API_KEY,
    restRoot: "https://api.lightrail.com/v1/",
    sharedSecret: process.env.LIGHTRAIL_SHARED_SECRET
});
const shopperToken = lightrail.generateShopperToken({shopperId: "customer-id-from-your-system"});
```
Note, the redemption and account balance widgets must be on authenticated pages as they require a `shopperId`.
You may decide whether you'd like your customers to be signed in to purchase gift cards. 
If you'd like to allow gift card purchase from an unauthenticated page simply generate a Shopper Token with `shopperId: ""`.

## Support
Looking for an example? Check out our [sample app](https://github.com/Giftbit/stripe-integration-sample-webapp) which is a working example of the entire Drop-in Gift Card solution.

Contact us any time at hello@lightrail.com —- we are here to help.