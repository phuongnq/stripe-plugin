# CrafterCMS Stripe Plugin

A Stripe plugin which uses checkout for subscriptions. This plugin is a modified version of [stripe-samples/checkout-single-subscription](https://github.com/stripe-samples/checkout-single-subscription) for CrafterCMS Studio.

A sample CrafterCMS Project uses this plugin: https://github.com/phuongnq/craftercms-stripe-sample-project

# Installation

1. Install the plugin via Studio's Plugin Management UI under `Site Tools` > `Plugin Management`

   OR You can also install this plugin by cloning this repository and using the Studio API.

    1a. Create a Studio JWT Token.

    1b. Execute the following CURL command a terminal

    ```bash
    curl --location --request POST 'http://SERVER_AND_PORT/studio/api/2/marketplace/copy' \
    --header 'Authorization: Bearer THE_JWT_TOKEN_FOR_STUDIO' \
    --header 'Content-Type: application/json' \
    --data-raw '{
      "siteId": "YOUR-PROJECT-ID",
      "path": "THE_ABSOLUTEL_FILE_SYSTEM_PATH_TO_THIS_REPO",
      "parameters": { }
    }
    ```

    OR You can aslo install this plugin by cloning this repository and using [Crafter CLI Commands](https://docs.craftercms.org/en/4.0/new-ia/reference/devcontentops-toolkit/copy-plugin.html)

Values starting with `${enc:...}` are encrypted text using [CrafterCMS Encryption Tool](https://docs.craftercms.org/en/4.0/system-administrators/activities/authoring/main-menu-encryption-tool.html#encryption-tool)

# Using Checkout for subscriptions

**1. Create a new Stripe Prices Page**

![Stripe Page](/stripe_prices_page.png)

Enter your prices plans. For example, by default the plugin has 3 plans: `basicPrice`, `proPrice`, `enterprisePrice`.

* basicPrice:
    * Title: Starter
    * Thumbnail: /static-assets/plugins/org/craftercms/plugin/stripe/img/starter.png
    * Price: $12
    * Price Name: basicPrice (**Note: Price name is fixed to retrieve from server API `delivery/scripts/rest/plugins/org/craftercms/plugin/stripe/config.get.groovy`)
    * recurring: month
* proPrice:
    * Title: Professional
    * Thumbnail: /static-assets/plugins/org/craftercms/plugin/stripe/img/professional.png
    * Price: $18
    * Price Name: proPrice (**Note: Price name is fixed to retrieve from server API `delivery/scripts/rest/plugins/org/craftercms/plugin/stripe/config.get.groovy`)
    * recurring: month
* enterprisePrice:
    * Title: Enterprise
    * Thumbnail: /static-assets/plugins/org/craftercms/plugin/stripe/img/enterprise.webp
    * Price: $250
    * Price Name: enterprisePrice (**Note: Price name is fixed to retrieve from server API `delivery/scripts/rest/plugins/org/craftercms/plugin/stripe/config.get.groovy`)
    * recurring: year

![Stripe Page Input](/stripe_prices_page_input.png)

After created, you can alson use the [Experience Builder](https://docs.craftercms.org/en/4.0/developers/experience-builder.html) functionality to Edit the page.

![Stripe Page Edit](/stripe_prices_page_edit.png)

**2. Use the Stripe CLI to create equivalent products from step 1**

[Checkout](https://stripe.com/docs/payments/checkout) is a pre-built payment page that lets you accept cards and Apple Pay. [Billing](https://stripe.com/docs/billing) is a suite of APIs that lets you model complex subscription plans. You can combine the two products to get a subscription payment page up and running without the need of a server.

When your customer is ready to pay, use [Stripe.js](https://stripe.com/docs/js) with the ID of your [Checkout Session](https://stripe.com/docs/api/checkout/sessions/object) to redirect them to your Checkout page.

**Demo**

The demo is running in test mode -- use `4242424242424242` as a test card number with any CVC + future expiration date.

Use the `4000002500003155` test card number to trigger a 3D Secure challenge flow.

Read more about testing on Stripe at https://stripe.com/docs/testing.

For more features see the [Checkout documentation](https://stripe.com/docs/payments/checkout/subscriptions).

The integration uses the [Checkout Sessions API](https://stripe.com/docs/api/checkout/sessions) for additional functionality.

**2. Create Products and Prices on Stripe**

This sample requires three [Price](https://stripe.com/docs/api/prices/object) IDs to create the Checkout page. Products and Prices are objects on Stripe that let you model a subscription.

**Using the Stripe CLI**

Create basic product
```sh
./stripe products create --name="Basic" --description="Basic plan"
```

Create premium product
```sh
./stripe products create --name="Premium" --description="Premium plan"
```

Create enterprise product
```sh
./stripe products create --name="Enterprise" --description="Enterprise plan"
```

Take note of the id value for the products you just created as you will need this to create prices. For example:
```json
{
  "id": "price_RANDOM_ID_VALUE"
}
```

Create price for Basic product, substituting `ID_OF_BASIC_PRODUCT` with the appropriate product Id:
```sh
./stripe prices create \
  -d "product=ID_OF_BASIC_PRODUCT" \
  -d "unit_amount=1200" \
  -d "currency=usd" \
  -d "recurring[interval]=month"
```

Create price for Premium product, substituting `ID_OF_PREMIUM_PRODUCT` with the appropriate product Id:
```sh
./stripe prices create \
  -d "product=ID_OF_PREMIUM_PRODUCT" \
  -d "unit_amount=1800" \
  -d "currency=usd" \
  -d "recurring[interval]=month"
```

Create price for Enterprise product, substituting `ID_OF_ENTERPRISE_PRODUCT` with the appropriate product Id:
```sh
./stripe prices create \
  -d "product=ID_OF_ENTERPRISE_PRODUCT" \
  -d "unit_amount=25000" \
  -d "currency=usd" \
  -d "recurring[interval]=year"
```

![Stripe CLI Commands](/stripe_cli_sample_commands.png)

  More Information: [Docs - Update your Products and Prices](https://stripe.com/docs/tax/checkout#product-and-price-setup)
</details>

**Using the Dashboard**

You can create Products and Prices [in the dashboard](https://dashboard.stripe.com/products). Create three recurring Prices to run this sample.

**Update your site-config.xml**

From Studio UI, open *Project Tools* > *Configuration* > *Engine Project Configuration*, Add your Stripe credentials and account information to `site-config.xml`:

```xml
<site>
    <version>4.0.1</version>
    <stripe>
      <publishableKey>${enc:CCE-V1#bUhsL6BI...}</publishableKey>
      <secretKey>${enc:CCE-V1#z48hAvk...}</secretKey>
      <webhookSecret>${enc:CCE-V1#sO73Bk+b...}</webhookSecret>
      <basicPriceId>price_1Mi...</basicPriceId>
      <proPriceId>price_1Mi...</proPriceId>
      <enterprisePriceId>price_1Mi...</enterprisePriceId>
      <callbackDomain>http://localhost:8080</callbackDomain>
    </stripe>
</site>
```

where:

* publishableKey: Stripe [publishable key](https://stripe.com/docs/keys#obtain-api-keys)
* secretKey: Stripe [secret key](https://stripe.com/docs/keys#obtain-api-keys)
* webhookSecret (optional): Webhook Secret
* basicPriceId: basicPrice ID
* proPriceId: proPrice ID
* enterprisePriceId: enterprisePrice ID
* callbackDomain: Callback domain of your application

**3. Open your plans page and try subscribing a plan**

![Stripe Subscribe](/stripe_subscribe.png)



